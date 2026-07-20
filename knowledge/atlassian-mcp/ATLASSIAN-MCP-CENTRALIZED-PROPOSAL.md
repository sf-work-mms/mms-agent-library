# Atlassian Rovo MCP — Centralized Deployment Proposal

**Date**: July 20, 2026  
**Author**: Saúl Fernández (Platform Engineering)  
**Status**: Proposal for IT Security & Atlassian Admin Approval  
**Audience**: IT Security, Atlassian Admin, Platform Engineering

---

## Executive Summary

MMS Engineering Platform is currently attempting to integrate **Atlassian Rovo MCP** (Model Context Protocol) with our centralized **MCP Catalog** deployed on Azure. Our goal is to standardize AI agent access to Jira Cloud, Confluence, and JSM through a single, audited proxy.

**Current Blocker**: The official Atlassian Rovo endpoint (`mcp.atlassian.com`) is not whitelisted by corporate Atlassian policies.

**Proposed Solution**: Deploy a **centralized MCP proxy** in MMS infrastructure that acts as a bridge. This proxy:
- ✅ Sits between our MCP Catalog and Atlassian Cloud
- ✅ Centralizes OAuth 2.1 authentication
- ✅ Provides audit logging for compliance
- ✅ Gets whitelisted instead of the official endpoint

**Parallel Requirement**: Provision a **service account** (API key) for programmatic Atlassian access by MMS services.

---

## Current State: Why Direct Connection Fails

```
Current Attempt:
┌─ MCP Catalog (Azure) ──────────┐
│ Configured with:               │
│ https://mcp.atlassian.com/...  │
└────────────┬────────────────────┘
             │ HTTPS request
             ↓
   ❌ BLOCKED by corporate whitelist
   (Reason: Official endpoint not pre-approved)
```

**Symptom**: Tools like `jira_search()`, `confluence_search()` fail with authentication/whitelist errors.

**Root Cause**: Atlassian Cloud admin has not whitelisted `mcp.atlassian.com`. New external endpoints require pre-approval.

---

## Proposed Solution: Centralized MCP Proxy

### Architecture

```
┌──────────────────────────────────────┐
│ MCP Catalog (Azure)                  │
│ - Located in                         │
│   terraform-epf-engineering-platform │
│ - Configured with proxy URL          │
└────────────┬─────────────────────────┘
             │ HTTPS (MCP Protocol)
             │ 
┌────────────▼──────────────────────────┐
│ MCP Proxy (MMS Infrastructure)        │
│ ✓ OAuth 2.1 Handler                   │
│ ✓ Token Management                    │
│ ✓ Audit Logging                       │
│ ✓ Request Forwarding                  │
└────────────┬──────────────────────────┘
             │ HTTPS (OAuth + MCP)
             │
┌────────────▼──────────────────────────┐
│ Approved Whitelist Entry              │
│ (IT Admin: whitelist proxy URL only)  │
└────────────┬──────────────────────────┘
             │
┌────────────▼──────────────────────────┐
│ Atlassian Cloud                       │
│ - Jira Cloud                          │
│ - Confluence                          │
│ - Jira Service Management             │
└───────────────────────────────────────┘
```

### How It Works

1. **MCP Catalog Configuration** (already exists)
   - Location: `terraform-epf-engineering-platform`
   - New entry: `atlassian-rovo: { type: "sse", url: "<proxy-url>" }`

2. **MCP Proxy** (GCP VM, ~€1-2K/month)
   - Receives MCP requests from Azure
   - Handles OAuth 2.1 token exchange with Atlassian
   - Forwards authenticated requests to Atlassian Cloud
   - Logs all access (audit trail)

3. **Whitelist Approval** (IT Admin action, one-time)
   - Whitelist the proxy URL instead of `mcp.atlassian.com`
   - Example: `https://mcp-proxy.mms.internal/atlassian`

4. **Result**
   - ✅ Tools work: `@atl jira_search()`, `@atl confluence_search()`, etc.
   - ✅ Centralized in MCP Catalog (no scattered integrations)
   - ✅ Audited: every call logged with user/agent identity
   - ✅ Standardized: consistent with other MCP integrations

---

## What We Need from IT & Atlassian Admin

### 1. Whitelist Approval (Atlassian Admin)

**Request**: Whitelist the MCP proxy URL in corporate Atlassian policies.

```
Whitelist Entry:
  URL: <proxy-url> (to be provided during deployment)
  Purpose: Centralized AI agent access to Jira/Confluence
  Auth: OAuth 2.1 (Atlassian standard)
  Scope: jira:read, confluence:read, issue:read
```

**Timeline**: 1-2 business days (once proxy is deployed with final URL)

**Security Justification**:
- OAuth 2.1 with PKCE (proof key for code exchange)
- TLS 1.3 encryption
- Audit logging on all tool calls
- Least-privilege scopes

---

### 2. Service Account (Atlassian Cloud)

**Purpose**: For programmatic access from MMS services (not MCP relay).

**Request**: Create a **service account** with:
- ✅ API key (for REST API calls)
- ✅ Scopes: `jira:read`, `jira:write`, `confluence:read`, `confluence:write`
- ✅ Access level: Limited to MMS projects (least privilege)

**Use Cases**:
- GitOps workflows trigger Jira updates
- Temporal workflows create Confluence pages
- GitHub Actions auto-comment on issues
- Incident response automation

**Difference from MCP Proxy**:
- MCP Proxy: OAuth relay for external AI clients
- Service Account: Direct API calls from MMS services

**Timeline**: 1-2 business days (parallel with whitelist approval)

---

## Implementation Timeline

| Phase | Duration | Deliverable | Approvals Needed |
|-------|----------|-------------|------------------|
| **Phase 1: Approval** | Week 1 | - Whitelist pre-approval from IT<br>- Service account request approval | IT Security, Atlassian Admin |
| **Phase 2: Deploy** | Week 2-3 | - MCP proxy running on GCP<br>- OAuth app registered<br>- Service account credentials delivered | None (IT signs off post-deploy) |
| **Phase 3: Test** | Week 3-4 | - MCP tools working: `@atl jira_search()`<br>- Audit logs verified<br>- API key tested with service account | Platform Engineering QA |
| **Phase 4: Enable** | Week 4-5 | - MCP Catalog updated<br>- Available to all platform engineers<br>- Documentation published | None (internal only) |

---

## Security & Compliance

### MCP Proxy Security

- **Authentication**: OAuth 2.1 with PKCE (Atlassian standard)
- **Encryption**: TLS 1.3 for all connections
- **Token Lifecycle**: Automatic refresh; 60-90 day rotation
- **Audit Logging**: Every MCP call logged with:
  - User/agent identity
  - Timestamp
  - Request/response summary
  - Status code

### Service Account Security

- **Credentials**: API key stored in GCP Secret Manager (encrypted at rest)
- **Least Privilege**: Limited to MMS projects only
- **IP Whitelisting**: Restricted to MMS service CIDR blocks
- **Audit Trail**: All API calls logged by Atlassian

### Compliance Alignment

- ✅ GDPR: Data access logged; retention policy enforced
- ✅ SOX: Audit trail for all changes
- ✅ Internal Policy: Least-privilege access control

---

## Deployment Architecture (Technical Details)

### MCP Proxy Stack

```
Component          | Technology      | Cost/Month
================== | =============== | ==========
Compute            | GCP VM (n1-std-2) | €80
Network            | Cloud Load Balancer | €15
Storage            | Persistent disk | €5
Secret Management  | GCP Secret Manager | <€1
Monitoring         | Cloud Logging | ~€10
Backup             | Cloud Snapshots | ~€10
================== | =============== | ===========
Total              |                 | ~€120/month
```

### Configuration (Terraform)

The MCP proxy will be configured in `terraform-epf-engineering-platform`:

```hcl
# Example: mcp-catalog/atlassian-rovo.tf

resource "google_compute_instance" "mcp_proxy_atlassian" {
  name         = "mcp-proxy-atlassian"
  machine_type = "n1-standard-2"
  zone         = "europe-west4-a"
  
  service_account_email = google_service_account.mcp_proxy.email
  
  # Security config, network, startup scripts, etc.
}

# MCP Catalog entry (already exists, just update URL)
locals {
  mcp_servers = {
    "atlassian-rovo" = {
      type      = "sse"
      url       = "https://${google_compute_instance.mcp_proxy_atlassian.network_interface[0].access_config[0].nat_ip}/mcp/atlassian"
      toolPrefix = "atl"
    }
  }
}
```

---

## Risks & Mitigation

| Risk | Mitigation |
|------|-----------|
| **Whitelist approval takes months** | Pre-align with IT/Atlassian admin this week. Show security model early. |
| **Token expiration causes outages** | Auto-refresh logic in proxy + monitoring alerts. |
| **Service account credentials compromise** | Secret rotation policy; IP whitelisting; audit alerts. |
| **Proxy performance bottleneck** | Load testing before GA; cache layer for frequently-accessed data. |

---

## Success Criteria

✅ **Week 2**: Whitelist + service account approved  
✅ **Week 3**: MCP proxy deployed and OAuth flow working  
✅ **Week 4**: MCP tools functional (`@atl jira_search()`, etc.)  
✅ **Week 5**: Available to all platform engineers  
✅ **Ongoing**: <1% error rate; <200ms latency (p95)  

---

## What We're NOT Asking For

❌ Atlassian admin role (we don't need it)  
❌ OAuth app pre-registration (Atlassian handles it)  
❌ Changes to firewall rules (just URL whitelisting)  
❌ New network infrastructure (uses existing Azure VNet)  

## What We ARE Asking For

✅ Whitelist the proxy URL in Atlassian cloud policies  
✅ Create a service account with API key + scopes  
✅ Sign off on security model (OAuth 2.1 + TLS 1.3 + audit logs)  

---

## Next Steps

### **This Week (Immediate)**

1. **IT Security**: Review this proposal; sign off on security model
2. **Atlassian Admin**: Confirm willingness to whitelist proxy URL + create service account
3. **Platform Engineering**: Finalize proxy deployment design

### **Week 1**

1. Submit whitelist request (with temporary proxy URL)
2. Request service account creation
3. Begin proxy deployment

### **Week 2-5**

1. Deploy proxy
2. Retrieve final URL
3. Submit final whitelist request
4. Test & validate
5. Rollout to team

---

## References

- [Atlassian Rovo MCP Official Docs](https://support.atlassian.com/atlassian-rovo-mcp-server/)
- [MCP Spec](https://modelcontextprotocol.io/)
- [MMS MCP Catalog](https://github.com/MediaMarktSaturn/terraform-epf-engineering-platform)
- [OAuth 2.1 Security Best Practices](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics)

---

## Contact & Questions

**Proposal Owner**: Saúl Fernández (Platform Engineering)  
**Questions?** Schedule a 30-min alignment call with:
- IT Security Lead
- Atlassian Admin
- Platform Engineering Lead

---

**Document Version**: 1.0  
**Status**: Ready for Submission  
**Next Review**: Post-whitelist approval

---

## 📍 Document Location

**Internal Location** (epft-docs):
- `docs/modules/ROOT/pages/team-internal/proposal_records/pr-008-atlassian-mcp-centralized.md`

This is an internal working document. For public/stakeholder materials, see:
- `BUSINESS-CASE-ATLASSIAN-MCP-PROXY.md` (business value, ROI)
- `PRESENTATION-GUIDE-ATLASSIAN-MCP.md` (how to present)
- `EXECUTIVE-SUMMARY-ATLASSIAN-MCP.md` (quick one-pager)

**GitHub**: https://github.com/MediaMarktSaturn/epft-docs/pull/150
