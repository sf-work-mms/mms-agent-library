# 🧪 Atlassian MCP Local Testing Setup

**For**: Saúl (@sarony)  
**Purpose**: Quick testing of Atlassian MCP with pi-coding-agent locally  
**Status**: Ready to Deploy  

---

## TL;DR — 5 Minute Setup

```bash
# 1. Create OAuth app in Atlassian Cloud (one-time)
#    Go to: Atlassian Admin Console → Security → OAuth 2.0 Applications
#    → Create app named "Atlassian MCP Local"
#    → Get Client ID + Secret

# 2. Setup wrapper
cd ~/repos/mms/code/mcp-atlassian-local
cp .env.example .env.local
# Edit .env.local with your Client ID + Secret

# 3. Run setup script
bash setup.sh

# 4. Start the server (Terminal 1)
source .env.local
bash start.sh

# 5. Start pi (Terminal 2)
pi

# 6. Test in pi
@atl jira_search(jql: "assignee = currentUser()")
```

---

## Detailed Setup

### Step 1: Create OAuth App in Atlassian Cloud

**Location**: Atlassian Cloud Admin Console

```
Settings
  → Security
    → OAuth 2.0 applications
      → Create application
```

**Fill in:**
- **App name**: `Atlassian MCP Local`
- **Redirect URIs**: `http://localhost:8001/callback`
- **Scopes**: 
  - `offline_access`
  - `jira:read`
  - `jira:write`
  - `confluence:read`
  - `confluence:write`

**Save these:**
- `Client ID`
- `Client Secret`

### Step 2: Configure Environment

```bash
cd ~/repos/mms/code/mcp-atlassian-local

# Copy template
cp .env.example .env.local

# Edit with your credentials
cat > .env.local << 'EOF'
ATLASSIAN_MCP_CLIENT_ID=your_client_id
ATLASSIAN_MCP_CLIENT_SECRET=your_client_secret
ATLASSIAN_MCP_PORT=8001
ATLASSIAN_MCP_HOST=127.0.0.1
EOF

# Protect it
chmod 600 .env.local
```

### Step 3: Install Dependencies & Test

```bash
cd ~/repos/mms/code/mcp-atlassian-local

# Run setup script (installs deps, creates systemd service optionally)
bash setup.sh

# Or manual install:
uv sync
```

### Step 4: Start the Server

**Terminal 1:**
```bash
cd ~/repos/mms/code/mcp-atlassian-local
source .env.local
bash start.sh
```

**Expected output:**
```
2026-07-14 14:15:30 - __main__ - INFO - 📋 Configuration loaded from environment
2026-07-14 14:15:30 - __main__ - INFO -    Client ID: YOUR_CLIENT_ID...
2026-07-14 14:15:30 - __main__ - INFO - 🚀 Starting Atlassian MCP Local Server on 127.0.0.1:8001
2026-07-14 14:15:31 - __main__ - INFO - ✅ Registered 5 tools: jira_search, jira_create, jira_update, confluence_search, confluence_create
```

### Step 5: Authenticate (First Run Only)

The server will prompt you to authenticate:

```
🔐 Initiating OAuth 2.1 flow with Atlassian
🌐 Opening browser for Atlassian OAuth authentication...

📍 If browser doesn't open, visit:
https://auth.atlassian.com/authorize?...

✋ Waiting for OAuth callback...
   After granting permission, copy the authorization code here.
   Authorization code: ← paste here
```

After granting permissions in browser:
1. Copy the auth code from the redirect URL
2. Paste it into the terminal
3. ✅ Token saved to `~/.config/atlassian-mcp/tokens.json`

### Step 6: Start pi & Test

**Terminal 2:**
```bash
pi
```

**Test in pi:**
```
@atl jira_search(jql: "assignee = currentUser()")

# Response:
Found 3 issues in EPDT project:
- EPDT-1234: Implement MCP integration
- EPDT-1235: Write deployment docs
- EPDT-1236: Test SSE protocol
```

---

## Available Tools

Once connected, use these via pi:

### Jira

```
@atl jira_search(jql: "text ~ \"MCP\" AND type = Task")
@atl jira_create(project: "EPDT", summary: "MCP test issue", issue_type: "Task")
@atl jira_update(issue_key: "EPDT-1234", fields: {"status": "In Review"})
```

### Confluence

```
@atl confluence_search(query: "MCP integration", space_key: "ENG")
@atl confluence_create(space_key: "ENG", title: "Test Page", body: "# Content")
```

---

## Troubleshooting

### Issue: "Connection refused"

```
ConnectionRefusedError: [Errno 111] Connection refused
```

**Solution:**
- [ ] Server running in Terminal 1? (`bash start.sh`)
- [ ] Port 8001 free? Check: `lsof -i :8001`
- [ ] Firewall blocking? `sudo ufw allow 8001` (if using ufw)

### Issue: "OAuth token not found"

```
❌ Error: No authorization code provided
```

**Solution:**
- Browser didn't open? Visit the URL manually
- Grant permissions
- Copy auth code from redirect
- Paste in terminal

### Issue: "Permission denied" on Jira query

```
Error calling jira_search: 401 Unauthorized
```

**Solution:**
- OAuth token expired? Server auto-refreshes (wait a moment)
- User doesn't have Jira access? Grant project access in Cloud
- Reauthorize: Delete `~/.config/atlassian-mcp/tokens.json` and restart

### Issue: No tools available

**Check:**
- [ ] Server started? See "✅ Registered 5 tools" in output
- [ ] pi detecting server? Check pi logs: `pi --debug`
- [ ] Correct tool prefix? Use `@atl` not `@atlassian`

---

## Next Steps

### After Testing Works:

1. **Document findings**:
   - What worked well?
   - What issues did you hit?
   - Performance observations?

2. **Report to team**:
   - Update [ATLASSIAN-MCP-DEPLOYMENT-PLAN.md](../docs/ATLASSIAN-MCP-DEPLOYMENT-PLAN.md) with testing results
   - Add screenshots of pi + Jira/Confluence integration

3. **Decide on Phase 1**:
   - Start full deployment (8-week plan)?
   - Iterate on local setup first?
   - Scale to team?

### Production Deployment:

For rolling out to all of MMS, follow the full 8-week plan:
[ATLASSIAN-MCP-DEPLOYMENT-PLAN.md](../docs/ATLASSIAN-MCP-DEPLOYMENT-PLAN.md)

---

## Architecture

```
┌─────────────────────────┐
│   pi-coding-agent       │
│   (your IDE/terminal)   │
└────────────┬────────────┘
             │
             │ MCP tools via SSE
             │ (http://localhost:8001)
             │
    ┌────────▼─────────────┐
    │ Atlassian MCP         │
    │ Local Wrapper         │
    │ (Python server)       │
    │                       │
    │ ✓ OAuth 2.1           │
    │ ✓ Token mgmt          │
    │ ✓ Tool proxy          │
    └────────┬─────────────┘
             │
             │ HTTPS + OAuth token
             │
      ┌──────▼──────────────┐
      │ Atlassian Cloud     │
      │ (Official MCP)      │
      │                     │
      │ ✓ Jira Cloud        │
      │ ✓ Confluence        │
      │ ✓ JSM               │
      └─────────────────────┘
```

---

## Code Structure

```
~/repos/mms/code/mcp-atlassian-local/
├── server.py              ← Main MCP server (OAuth + proxy logic)
├── pyproject.toml         ← Dependencies
├── start.sh              ← Quick start script
├── setup.sh              ← One-time setup
├── .env.example          ← Template
├── .env.local            ← YOUR credentials (ignored in git)
├── .gitignore            ← Security
└── README.md             ← Full documentation
```

---

## Security Checklist

- [ ] `.env.local` has `chmod 600` (read-only)
- [ ] `.env.local` NOT committed to git
- [ ] Token stored in `~/.config/atlassian-mcp/tokens.json` (NOT git)
- [ ] Only run on localhost (`127.0.0.1:8001`)
- [ ] Use `offline_access` scope only if needed (for refresh tokens)

---

**Questions?** Ask Saúl  
**Docs**: See [README.md](../code/mcp-atlassian-local/README.md) for full reference  
**Full Plan**: [ATLASSIAN-MCP-DEPLOYMENT-PLAN.md](../docs/ATLASSIAN-MCP-DEPLOYMENT-PLAN.md)
