# Atlassian MCP Integration

**Status**: Ready for Implementation  
**Owner**: Saúl (@sarony)  
**Updated**: 2026-07-15

---

## 📖 Documentation Index

### Quick Start
- **[ATLASSIAN-MCP-CORRECTED-DIRECT-CLOUD.md](./ATLASSIAN-MCP-CORRECTED-DIRECT-CLOUD.md)** ← **START HERE**
  - How to integrate Atlassian MCP directly with pi
  - 3-step setup (OAuth app, config, test)
  - No local server needed (cloud-native)

### Implementation Guides
- **[ATLASSIAN-MCP-DEPLOYMENT-PLAN.md](./ATLASSIAN-MCP-DEPLOYMENT-PLAN.md)**
  - Full 8-week deployment plan
  - Phase 1-5 breakdown
  - Risk analysis, success metrics, roadmap

- **[ATLASSIAN-MCP-QUICKSTART.md](./ATLASSIAN-MCP-QUICKSTART.md)**
  - For developers integrating Atlassian tools
  - Tool reference, examples, workflows
  - Troubleshooting

### Decision Documents
- **[ATLASSIAN-MCP-COMPARATIVE-ANALYSIS.md](./ATLASSIAN-MCP-COMPARATIVE-ANALYSIS.md)**
  - Why Atlassian Rovo MCP vs alternatives
  - Feature comparison matrix
  - Cost & operational analysis

- **[ATLASSIAN-MCP-CORRECTION-LOG.md](./ATLASSIAN-MCP-CORRECTION-LOG.md)**
  - My mistakes & lessons learned
  - Anti-sycophancy analysis
  - What I got wrong (over-engineering)

### Archived (Obsolete)
- `ATLASSIAN-MCP-LOCAL-SETUP-COMPLETE.md` — ~~Old wrapper approach~~ (ignore)
- `ATLASSIAN-MCP-LOCAL-TESTING.md` — ~~Local setup guide~~ (ignore)

---

## 🎯 Quick Reference

### Architecture (Correct Approach)

```
pi-coding-agent
       ↓
   @atl jira_search(...)
       ↓
https://mcp.atlassian.com/v1/mcp/authv2
       ↓
Atlassian Cloud (Jira, Confluence, JSM, Bitbucket, Compass)
```

### 3-Step Setup

1. **Create OAuth App** (Atlassian Cloud admin)
   ```
   Settings → Security → OAuth 2.0 → Create
   Name: "Atlassian MCP (pi-coding-agent)"
   Redirect: https://mcp.atlassian.com/callback
   Scopes: offline_access jira:* confluence:* jira-servicedesk:*
   ```

2. **pi Already Configured**
   ```json
   // In core-agent-library/mcp-global.json
   "atlassian-rovo": {
     "type": "sse",
     "url": "https://mcp.atlassian.com/v1/mcp/authv2",
     "toolPrefix": "atl"
   }
   ```

3. **First Test in pi**
   ```
   @atl jira_search(jql: "assignee = currentUser()")
   → Browser opens for OAuth
   → Authorize
   → ✅ Results appear
   ```

---

## 🛠️ Available Tools

Once authorized:

- **Jira**: `jira_search`, `jira_create`, `jira_update`, `jira_add_comment`
- **Confluence**: `confluence_search`, `confluence_create`, `confluence_update`
- **JSM**: `jsm_search_requests`, `jsm_create_request`
- **Bitbucket**: `bitbucket_search`
- **Compass**: `compass_search`

---

## 📊 Project Status

| Phase | Status | Timeline |
|-------|--------|----------|
| **Discovery & Setup** | ✅ Complete | Done |
| **Phase 1: Readiness** | ⏳ Not Started | Week 1-2 |
| **Phase 2: Configuration** | ⏳ Not Started | Week 3-4 |
| **Phase 3: Pilot** | ⏳ Not Started | Week 5-6 |
| **Phase 4: Hardening** | ⏳ Not Started | Week 7 |
| **Phase 5: GA** | ⏳ Not Started | Week 8 |

---

## 📞 Support

- **Questions?** See [ATLASSIAN-MCP-QUICKSTART.md](./ATLASSIAN-MCP-QUICKSTART.md)
- **Big decisions?** See [ATLASSIAN-MCP-COMPARATIVE-ANALYSIS.md](./ATLASSIAN-MCP-COMPARATIVE-ANALYSIS.md)
- **Full plan?** See [ATLASSIAN-MCP-DEPLOYMENT-PLAN.md](./ATLASSIAN-MCP-DEPLOYMENT-PLAN.md)
- **Contact**: Saúl (@sarony), Tachikoma (@tachikoma)

---

**Last Updated**: 2026-07-15  
**Repository**: https://github.com/sf-work-mms/mms-agent-library
