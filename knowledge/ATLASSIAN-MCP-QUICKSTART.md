# Atlassian Rovo MCP — Quick Start Configuration

**For**: MMS Engineering Platform  
**Created**: 2026-07-14

---

## TL;DR — What You Need to Know

✅ **Official MCP server by Atlassian** — battle-tested, GA status  
✅ **Cloud-hosted** — no self-hosting needed  
✅ **Real-time** — Server-Sent Events (SSE) protocol for live data  
✅ **Secure** — OAuth 2.1 + API token options, respects your Jira permissions  
✅ **Integrations**: Cursor, VS Code, Claude, ChatGPT, custom agents

**Endpoint**: `https://mcp.atlassian.com/v1/mcp/authv2`

---

## 1. Prerequisites (Before Week 1)

```bash
# ✅ MMS must have:
1. Active Atlassian Cloud tenant
   └─ Jira Cloud
   └─ Confluence Cloud  
   └─ Jira Service Management
   └─ OAuth 2.0 admin console access

2. Developer environment:
   └─ Cursor or VS Code installed
   └─ Claude for Teams account (optional)

3. GCP access:
   └─ Secret Manager to store OAuth credentials
   └─ Service Account for token rotation automation
```

---

## 2. Atlassian Cloud Setup (5 minutes)

### Step 1: Create OAuth App

Navigate to **Atlassian Cloud Admin Console**:
```
Settings → Security → OAuth 2.0 applications → Create application
```

**Configuration:**
| Field | Value |
|-------|-------|
| App name | `MMS Rovo MCP` |
| Application type | `Authorization code with PKCE` |
| Redirect URIs | `https://mcp.atlassian.com/callback` |

**Scopes** (minimum required):
```
✅ jira:read
✅ jira:write
✅ confluence:read
✅ confluence:write
✅ jira-servicedesk:read
✅ jira-servicedesk:write
```

**Save credentials:**
- Client ID: `[STORE IN GCP SECRET MANAGER]`
- Client secret: `[STORE IN GCP SECRET MANAGER]`

### Step 2: Store Credentials Securely

```bash
# Via gcloud CLI:
gcloud secrets create atlassian-mcp-client-id \
  --replication-policy="automatic" \
  --data-file=- <<< "YOUR_CLIENT_ID"

gcloud secrets create atlassian-mcp-client-secret \
  --replication-policy="automatic" \
  --data-file=- <<< "YOUR_CLIENT_SECRET"

# Verify:
gcloud secrets versions access latest --secret="atlassian-mcp-client-id"
```

---

## 3. IDE Setup

### Option A: Cursor (Recommended)

**One-Click Install:**
```
https://cursor.com/en/install-mcp?name=Atlassian-Rovo-MCP&config=eyJ1cmwiOiJodHRwczovL21jcC5hdGxhc3NpYW4uY29tL3YxL21jcC9hdXRodjIifQ%3D%3D
```

**Or Manual Config:**

Edit `~/.cursor/settings/extensions.json`:
```json
{
  "mcpServers": {
    "atlassian": {
      "url": "https://mcp.atlassian.com/v1/mcp/authv2",
      "type": "sse",
      "auth": "oauth2"
    }
  }
}
```

**In Cursor, press `Ctrl+K` and type:**
```
@jira create issue with title "Test issue" in project "PLATFORM"
```

### Option B: VS Code

**Install Extension:**
```bash
code --install-extension Atlassian.mcp-atlassian
```

**Or Edit settings manually:**
`~/.config/Code/User/settings.json`:
```json
{
  "mcp": {
    "servers": {
      "atlassian": {
        "url": "https://mcp.atlassian.com/v1/mcp/authv2",
        "type": "sse",
        "auth": "oauth2"
      }
    }
  }
}
```

**First use:** Click "Authenticate with Atlassian" button in sidebar  
→ Browser redirect to OAuth consent screen  
→ Grant permissions  
→ Token auto-stored locally

### Option C: Claude for Teams

```
1. Open Claude at claude.ai
2. Click "Settings" → "MCP Servers"
3. Paste: https://mcp.atlassian.com/v1/mcp/authv2
4. Click "Authenticate"
5. Grant OAuth permissions in browser redirect
```

---

## 4. First Tests

### Test 1: Search Jira

**In Cursor/Claude:**
```
@jira search for issues assigned to me with status "In Progress"
```

**Expected output:**
```
Found 3 issues:
- EPDT-1234: Implement MCP integration
- EPDT-1235: Write deployment docs
- EPDT-1236: Test SSE protocol
```

### Test 2: Create Confluence Page

```
@confluence create page titled "MCP Integration Guide" 
in space "ENG" with content "## Getting Started..."
```

### Test 3: Link Issue to Confluence

```
@jira update issue EPDT-1234 
set description to "Related docs: https://confluence/..."
```

---

## 5. Token Management

### OAuth Token Refresh

**Automatic**: MCP client refreshes tokens automatically within the SSE stream (no interruption)

### API Token Management (if using API tokens instead of OAuth)

**Generate token in Atlassian Cloud:**
```
Settings → Personal settings → API tokens → Create API token
```

**Store safely:**
```bash
gcloud secrets create atlassian-mcp-api-token \
  --data-file=- <<< "YOUR_API_TOKEN"
```

**Rotation schedule**: **Every 90 days** (or quarterly)

```bash
# Automated via GitHub Actions:
.github/workflows/rotate-atlassian-tokens.yml
```

---

## 6. Security Checklist

| Item | Status | Notes |
|------|--------|-------|
| OAuth app created in Atlassian Cloud | ⬜ | Only admin access |
| Client ID/secret stored in GCP Secret Manager | ⬜ | Never commit to git |
| API scopes minimized (read/write only what needed) | ⬜ | No admin scopes |
| MCP clients configured on developer machines | ⬜ | Via Cursor/VS Code settings |
| Token rotation schedule established | ⬜ | Quarterly minimum |
| Audit logging enabled in Atlassian Cloud | ⬜ | Monitor API access |
| Incident runbook created | ⬜ | Steps if token leaked |

---

## 7. MCP Tools Available

### Jira Tools
```
- jira_search(jql: string) → Issue[]
- jira_create(project: string, summary: string, ...) → Issue
- jira_update(issueKey: string, fields: dict) → Issue
- jira_add_comment(issueKey: string, comment: string) → Comment
- jira_transition(issueKey: string, toStatus: string) → Issue
```

### Confluence Tools
```
- confluence_search(query: string, space: string) → Page[]
- confluence_create_page(space: string, title: string, body: string) → Page
- confluence_update_page(pageId: string, body: string) → Page
- confluence_get_page(pageId: string) → Page
```

### Jira Service Management Tools
```
- jsm_search_requests(query: string) → Request[]
- jsm_create_request(projectKey: string, issueTypeId: string, ...) → Request
- jsm_get_request(requestId: string) → Request
- jsm_add_comment(requestId: string, comment: string) → Comment
```

---

## 8. Troubleshooting

### Issue: "Authentication failed"

```
1. Check credentials in GCP Secret Manager
2. Verify OAuth app client ID matches
3. Re-authorize in Cursor/Claude
4. Check Atlassian Cloud admin logs: Settings → Audit log → OAuth events
```

### Issue: "MCP connection timeout"

```
1. Check network: curl -v https://mcp.atlassian.com/v1/mcp/authv2
2. Verify no firewall blocking (corporate proxy?)
3. Check Atlassian status: status.atlassian.com
4. Restart IDE MCP connection
```

### Issue: "Permission denied on Jira query"

```
1. Verify user has permission on the project
2. Check Jira project settings: Who can view issues?
3. Query your own issues first: @jira search assigned:me
4. OAuth scope may need expansion (re-authorize)
```

### Issue: "Token expired"

```
1. OAuth: Auto-refresh happens in background (no action needed)
2. API Token: Generate new token, update in Secret Manager
3. Restart IDE
```

---

## 9. Common Workflows

### Workflow A: Auto-Link GitHub PR to Jira

**In Cursor (or GitHub Action):**
```
@jira update EPDT-1234 
set "GitHub PR": "https://github.com/MediaMarktSaturn/repo/pull/42"
```

### Workflow B: Create Issue from Meeting Notes

```
@jira create issue in PLATFORM
summary: "AI summary of meeting notes"
description: "See linked Confluence page for full notes"

@confluence link to issue EPDT-1234
```

### Workflow C: Generate Status Report

```
@jira search project:PLATFORM 
status:"In Progress" OR status:"In Review"

@confluence create page titled "Weekly Status YYYY-MM-DD"
body: [Consolidated summary from Jira + Confluence]
```

---

## 10. Rollout Checklist

- [ ] Phase 1: Pilot (5-10 developers)
- [ ] Phase 2: Feedback collection (1 week)
- [ ] Phase 3: Rollout to all engineers
- [ ] Phase 4: Optional: Integrate into internal agents

---

## Support & Escalation

| Issue | Action |
|-------|--------|
| **MCP client won't connect** | Check network, restart IDE, verify creds |
| **Jira permissions issue** | Contact Jira admin, check project access |
| **Token/auth issue** | Check GCP Secret Manager, regenerate OAuth token |
| **Atlassian outage** | Check `status.atlassian.com`, wait for recovery |
| **MCP server down** | Escalate to Atlassian support (SLA: 99.5%) |

---

**Questions?** Ask in #mms-platform Slack or ping Saúl @sarony

---

Generated: 2026-07-14  
Reference: [Full Deployment Plan](./ATLASSIAN-MCP-DEPLOYMENT-PLAN.md)
