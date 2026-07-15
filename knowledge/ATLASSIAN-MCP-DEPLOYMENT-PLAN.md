# Atlassian Rovo MCP Server — Deployment Plan for MMS

**Status**: Planning Phase  
**Date**: 2026-07-14  
**Owner**: Saúl (@sarony)  
**Domain**: MMS Engineering Platform  

---

## 1. Executive Summary

This document outlines a **phased deployment plan** for integrating the **official Atlassian Rovo MCP Server** into MMS's engineering platform following the migration from Atlassian Server to Cloud.

### Key Objectives:
- Integrate Jira Cloud, Confluence, and JSM with MCP-compatible AI tools (Claude, ChatGPT, Cursor, VS Code)
- Enable real-time data access via Server-Sent Events (SSE)
- Maintain strict security posture (OAuth 2.1, least privilege, RBAC alignment)
- Support agentic workflows for MMS automation

### Timeline: 6-8 weeks
### Blast Radius: Engineering Platform team + Optional expansion to product teams

---

## 2. Current State Analysis

### 2.1 Migration Context
```
Atlassian Server (EOL)  →  Atlassian Cloud (Active)
- Legacy self-hosted    →  Managed cloud service
- Manual auth mgmt       →  OAuth 2.1 / API tokens
- Limited integrations   →  MCP-native integrations
```

### 2.2 MMS Infrastructure Prerequisites
- ✅ Atlassian Cloud tenant active (Jira, Confluence, JSM)
- ✅ OAuth 2.0 support in MMS identity layer (Okta/SSO)
- ⚠️ **TODO**: Verify Atlassian Cloud OAuth apps created in admin console
- ⚠️ **TODO**: Document current API token policies

### 2.3 Current MMS Tooling
- **IDE Integration**: Cursor, VS Code (dev team)
- **CI/CD**: GitHub Actions (MR/PR integration)
- **AI Clients**: Claude for Teams, internal agents
- **MCP Registry**: Not yet integrated centrally

---

## 3. Atlassian Rovo MCP Server: Technical Overview

### 3.1 What It Is
A **cloud-hosted Model Context Protocol server** maintained by Atlassian that:
- Acts as a bridge between Atlassian Cloud and external AI tools
- Exposes Jira, Confluence, JSM, Bitbucket, Compass via MCP tools
- Uses OAuth 2.1 for secure, user-respecting authentication
- Communicates via Server-Sent Events (SSE) for real-time data sync

### 3.2 Supported Operations

#### Jira Cloud:
- Search, summarize, filter issues
- Create, update issues from natural language
- Link PRs to issues
- Bulk comment operations
- Project/sprint queries

#### Confluence:
- Search, summarize pages
- Create, update pages
- Link issues to pages
- Query page tree hierarchy

#### Jira Service Management:
- Ticket lifecycle management
- Request search & routing
- SLA tracking

#### Bitbucket:
- Repository metadata
- Branch/PR queries
- Code search context

#### Compass:
- Component registry queries
- Dependency graphs

### 3.3 Authentication Options

| Method | Use Case | Security | Token Expiry |
|--------|----------|----------|--------------|
| **OAuth 2.1** | Humans in IDEs, AI assistants | ⭐⭐⭐⭐⭐ Highest | Session-based |
| **API Token** | Automated services, agents | ⭐⭐⭐⭐ High | Manual rotation |
| **PAT (Personal Access Token)** | Dev individual use | ⭐⭐⭐ Medium | ~1 year |

**Recommendation for MMS**: 
- Primary: **OAuth 2.1** for interactive tools (Cursor, VS Code, Claude for Teams)
- Secondary: **API Tokens** (rotated quarterly) for internal agents & automation

### 3.4 SSE Protocol Configuration

**Endpoint**: `https://mcp.atlassian.com/v1/mcp/authv2`

**Connection Flow**:
```
┌─────────────────────┐
│  MCP Client         │
│  (Cursor/Claude)    │
└──────────┬──────────┘
           │
           │ HTTP(S) SSE connection
           │
      ┌────▼─────────────────────────┐
      │  Atlassian Rovo MCP Server    │
      │  https://mcp.atlassian.com    │
      └──────────┬────────────────────┘
                 │
                 │ Secure OAuth2.1 tokens
                 │
         ┌───────▼──────────┐
         │  Atlassian Cloud │
         │ (Jira, Confluence)
         └──────────────────┘
```

---

## 4. Deployment Phases

### **PHASE 1: Discovery & Readiness (Week 1-2)**

**Goal**: Understand current state, prepare infrastructure, and gain stakeholder alignment.

#### 1.1 Infrastructure Audit
- [ ] Verify Atlassian Cloud admin console access
- [ ] List all Jira projects, Confluence spaces, JSM instances
- [ ] Document current OAuth 2.0 apps registered in Atlassian
- [ ] Review existing API token policies
- [ ] Verify GCP KMS/Secret Manager can store MCP tokens securely

**Owners**: MMS Platform Team + Cloud Ops  
**Deliverable**: Atlassian Cloud Inventory Spreadsheet

#### 1.2 Team Alignment
- [ ] Review this plan with Saúl & Jira admins
- [ ] Identify pilot users (Cursor/Claude early adopters)
- [ ] Define success criteria (e.g., "2 MCP integrations, 50% team adoption by Week 6")
- [ ] Clarify scope: Phase 1 = MMS internal only (not customer-facing)

**Owner**: Saúl  
**Deliverable**: Stakeholder sign-off

#### 1.3 Security Review
- [ ] Review Atlassian's MCP security docs
- [ ] Validate OAuth 2.1 compliance with MMS security policy
- [ ] Plan token rotation schedule
- [ ] Document RBAC mapping: MMS teams → Atlassian projects
- [ ] Create incident response playbook (token compromise, data access abuse)

**Owners**: MMS Security + Platform Team  
**Deliverable**: Security Approval Document

### **PHASE 2: Configuration & Setup (Week 3-4)**

**Goal**: Configure OAuth apps, prepare MCP deployment infrastructure, and test connectivity.

#### 2.1 Atlassian Cloud Configuration
```bash
# In Atlassian Cloud Admin Console:
1. Navigate to: Settings → Security → OAuth 2.0 applications
2. Create new OAuth app: "MMS Rovo MCP"
   - Redirect URI: https://mcp.atlassian.com/callback
   - Scopes: Jira (read/write), Confluence (read/write), JSM (read/write)
3. Generate client ID + secret
4. Store in GCP Secret Manager:
   - secret: `atlassian-mcp-client-id`
   - secret: `atlassian-mcp-client-secret`
```

**Deliverable**: 
- OAuth app ID + credentials in Secret Manager
- Confirmation screenshot of admin panel

#### 2.2 MMS MCP Registry Entry
- [ ] Create entry in `core-resource-catalog` for Atlassian MCP service
- [ ] Document endpoint: `https://mcp.atlassian.com/v1/mcp/authv2`
- [ ] Record supported tools: Jira, Confluence, JSM, Bitbucket, Compass
- [ ] Link to service documentation

**Location**: `core-resource-catalog/services/mcp/atlassian-rovo.yaml`

#### 2.3 MCP Client Configuration Templates

**For Cursor:**
```json
{
  "mcpServers": {
    "atlassian": {
      "url": "https://mcp.atlassian.com/v1/mcp/authv2",
      "type": "sse",
      "auth": "oauth2",
      "clientId": "${ATLASSIAN_MCP_CLIENT_ID}",
      "clientSecret": "${ATLASSIAN_MCP_CLIENT_SECRET}"
    }
  }
}
```

**For VS Code:**
```json
{
  "mcp": {
    "servers": {
      "atlassian": {
        "command": "node",
        "args": ["./atlassian-mcp-client.js"],
        "env": {
          "MCP_SERVER": "https://mcp.atlassian.com/v1/mcp/authv2"
        }
      }
    }
  }
}
```

**For Claude for Teams:**
- Use one-click install link: https://claude.ai/install-mcp?name=Atlassian-Rovo-MCP&config=...
- OAuth redirect to user's Atlassian Cloud

#### 2.4 Testing Checklist
- [ ] Test OAuth flow (Cursor + Claude for Teams)
- [ ] Verify Jira issue search via MCP tool
- [ ] Verify Confluence page creation via MCP tool
- [ ] Test with different Atlassian project permissions (RBAC)
- [ ] Verify SSE connection stability (60+ min idle test)
- [ ] Validate token refresh behavior

**Deliverable**: Test Report + Screenshots

### **PHASE 3: Pilot Program (Week 5-6)**

**Goal**: Validate MCP in real workflows with early adopters; identify edge cases.

#### 3.1 Pilot Cohort Selection
- [ ] Recruit 5-10 early adopters from engineering team
- [ ] Mix: Frontend, backend, DevOps, platform engineers
- [ ] Provide them with Cursor/Claude with MCP pre-configured

#### 3.2 Pilot Use Cases

**Use Case A: Issue Auto-Linking**
- Developer works in Cursor on a feature branch
- MCP auto-links Jira issue from branch name
- Developer updates issue status via MCP tool within IDE

**Use Case B: Confluence Documentation**
- Product manager creates RFC in Confluence
- Linked Jira ticket auto-populated from RFC context
- AI agent summarizes RFC for stakeholders

**Use Case C: JSM Ticket Triage**
- Support team uses Claude + Atlassian MCP
- Claude reads incoming support tickets
- AI auto-suggests priority/assignment based on Jira project templates

#### 3.3 Feedback Collection
- [ ] Weekly sync with pilot cohort
- [ ] Google Form feedback (5-question survey)
- [ ] Track MCP tool usage metrics (# calls, errors, latency)
- [ ] Document any access control issues

**Deliverable**: Pilot Results Report

### **PHASE 4: Hardening & Automation (Week 7)**

**Goal**: Prepare production deployment; build internal automation.

#### 4.1 Build MCP Client Wrappers
```python
# New file: mms/code/mcp-atlassian-wrapper/
# Wraps Atlassian MCP for internal agent use

from mcp_client import MCPClient

class AtlassianMCPClient:
    def __init__(self, oauth_token: str):
        self.client = MCPClient(
            url="https://mcp.atlassian.com/v1/mcp/authv2",
            auth=oauth_token
        )
    
    def search_jira(self, jql: str) -> list:
        """Search Jira using MCP tool"""
        return self.client.call_tool("jira_search", {"jql": jql})
    
    def create_issue(self, project: str, issue_data: dict) -> str:
        """Create Jira issue from natural language"""
        return self.client.call_tool("jira_create", {
            "project": project,
            "data": issue_data
        })
    
    def search_confluence(self, query: str) -> list:
        """Search Confluence pages"""
        return self.client.call_tool("confluence_search", {"query": query})
```

#### 4.2 Token Rotation Automation
```bash
# GitHub Actions workflow: .github/workflows/mcp-token-rotation.yml

name: Rotate Atlassian MCP Tokens
on:
  schedule:
    - cron: '0 2 1 * *'  # First of month, 2 AM UTC

jobs:
  rotate:
    runs-on: ubuntu-latest
    steps:
      - name: Rotate API Token
        run: |
          # 1. Generate new token in Atlassian Cloud
          # 2. Store in GCP Secret Manager
          # 3. Notify team via Slack
```

#### 4.3 Monitoring & Alerting
- [ ] CloudWatch logs for MCP SSE connections
- [ ] Alert on: auth failures, high latency (>5s), connection drops
- [ ] Dashboard: MCP usage by team, error rates, token expiry

**Location**: `mms/gitops/engineering-platform-gitops/monitoring/atlassian-mcp/`

#### 4.4 Documentation
- [ ] Write operator manual: `mms/docs/ATLASSIAN-MCP-OPERATIONS.md`
- [ ] Create user quickstart guide for Cursor/VS Code setup
- [ ] Document supported tools with examples
- [ ] Incident runbook: "MCP token compromised"

### **PHASE 5: General Availability (Week 8)**

**Goal**: Roll out to all engineering teams; optimize based on feedback.

#### 5.1 Rollout
- [ ] Deploy to all dev machines via MDM profile (if applicable)
- [ ] Create Slack bot to guide setup
- [ ] Schedule onboarding sessions (30 min each)
- [ ] Track adoption metrics

#### 5.2 Expand Scope (Optional)
- [ ] Add MMS Bitbucket integration (PR linking)
- [ ] Integrate with internal agent orchestration
- [ ] Build Jira-to-GitHub automation workflows
- [ ] Connect to product analytics (Compass)

---

## 5. Risk & Mitigation

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|-----------|
| **OAuth token compromise** | High data exposure | Low | Quarterly rotation, audit logs, alerts on unusual access |
| **MCP service outage** | IDE workflows blocked | Low | Atlassian SLA 99.5%, graceful fallback to web UI |
| **Permission escalation bug** | Unauthorized data access | Very Low | MCP respects user RBAC; pre-auth testing on pilot cohort |
| **Performance degradation** | Slow IDE experience | Medium | Monitor SSE latency, cache frequently accessed data |
| **Integration complexity** | Team confusion | Medium | Clear docs, onboarding sessions, internal support channel |

---

## 6. Success Criteria

### Phase-Gate Metrics

| Phase | Success Metric | Target | Acceptance |
|-------|---|---|---|
| **Phase 1** | Stakeholder sign-off | 100% | Saúl + Jira admin approval |
| **Phase 2** | Successful OAuth + SSE test | 100% | All test checklist items pass |
| **Phase 3** | Pilot adoption rate | ≥50% | ≥5/10 pilots using MCP weekly |
| **Phase 3** | Error rate | <1% | <1 error per 100 MCP calls |
| **Phase 5** | Team adoption | ≥30% | ≥30% of eng team configured MCP |
| **Phase 5** | User satisfaction | ≥4/5 | NPS score from feedback survey |

---

## 7. Technical Spike: SSE Protocol Deep Dive

### 7.1 SSE vs Traditional HTTP

| Aspect | SSE | HTTP Polling |
|--------|-----|------|
| Latency | Real-time (<100ms) | 30-60s poll interval |
| Bandwidth | Efficient (streaming) | Higher (repeated requests) |
| Server load | Lower (single connection) | Higher (many requests) |
| Complexity | Moderate | Simple |

### 7.2 SSE Reconnection Strategy

Atlassian MCP handles reconnections automatically:
```
Client connects → SSE open
  ↓
Server sends tools list + auth challenge
  ↓
Client authenticates via OAuth
  ↓
Stream of tool calls/responses
  ↓
Connection closed → Auto-reconnect with exponential backoff
```

### 7.3 Security in SSE Transport

- ✅ HTTPS only (encrypted TLS 1.3)
- ✅ OAuth token in Authorization header (not in URL)
- ✅ Token refresh within stream (no interruption)
- ✅ No sensitive data in SSE event payload (indirect references only)

---

## 8. Rollback Plan

If issues arise:

### Minor Issues (High latency, occasional errors)
```
→ Clear client cache
→ Restart IDE MCP connection
→ Check Secret Manager token validity
```

### Major Issues (Auth failures, data corruption)
```
→ Disable MCP servers in Atlassian Cloud admin console
→ Alert team: "MCP temporarily disabled for investigation"
→ Investigate logs in Cloud Audit Trail
→ Rotate OAuth credentials
→ Resume Phase 3 testing before re-enabling
```

### Complete Rollback
```
→ Remove MCP configs from all clients
→ Revoke OAuth app in Atlassian Cloud
→ Notify stakeholders: "MCP program paused"
→ Post-mortem + decision on restart timeline
```

---

## 9. Roadmap: Future Enhancements

### 9.1 Internal Agents (Q3 2026)
```
MMS Internal Agents + Atlassian MCP:
- Automated issue creation from pull requests
- Confluence auto-documentation from code changes
- JSM ticket routing via ML classification
```

### 9.2 Developer Experience (Q3 2026)
```
- Slack bot: "/jira search epic:PLATFORM" via MCP
- GitHub bot: Auto-link Jira issues in PR descriptions
- VS Code extension: Jira + Confluence sidebar in IDE
```

### 9.3 Analytics & Insights (Q4 2026)
```
- Dashboard: Team MCP usage trends
- Compliance: Audit trail of all MCP access
- Optimization: Heat map of Jira/Confluence queries
```

---

## 10. Resources & Links

### Official Docs
- [Atlassian Rovo MCP Server](https://github.com/atlassian/atlassian-mcp-server)
- [Getting Started Guide](https://support.atlassian.com/atlassian-rovo-mcp-server/docs/getting-started-with-the-atlassian-remote-mcp-server/)
- [Supported Tools](https://support.atlassian.com/atlassian-rovo-mcp-server/docs/supported-tools/)
- [Security & Access Policies](https://support.atlassian.com/security-and-access-policies/docs/understand-atlassian-rovo-mcp-server/)
- [SSE Protocol Documentation](https://support.atlassian.com/atlassian-rovo-mcp-server/docs/server-sent-events-protocol/)

### MMS Internal
- MMS Atlassian Cloud Admin: [Link to admin console]
- MCP Token Storage: `gcp-secret-manager:atlassian-mcp-*`
- Issue Tracking: EPDT-[TBD] (to be created)

---

## 11. Sign-Off

| Role | Name | Date | Approval |
|------|------|------|----------|
| **Project Lead** | Saúl | — | ⬜ |
| **Platform Lead** | [Name] | — | ⬜ |
| **Security Lead** | [Name] | — | ⬜ |
| **Jira Admin** | [Name] | — | ⬜ |

---

**Document Version**: 1.0  
**Last Updated**: 2026-07-14  
**Next Review**: 2026-07-21
