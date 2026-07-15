# MMS Agent Library — Knowledge Base

**Domain**: MediaMarktSaturn (MMS) Engineering Platform  
**Purpose**: Centralized knowledge, documentation, and decision logs for MMS AI/Agent infrastructure  
**Owner**: Saúl (@sarony)

---

## 📚 Structure

```
mms-agent-library/
├── knowledge/              ← Knowledge base, docs, decision logs
│   ├── ATLASSIAN-MCP-*.md  ← Atlassian MCP integration docs
│   └── [other docs]
├── README.md               ← This file
└── .git/                   ← Version control
```

## 📖 Knowledge Base

### Atlassian MCP Integration

Location: `knowledge/atlassian-mcp/`

- **[ATLASSIAN-MCP-CORRECTED-DIRECT-CLOUD.md](./knowledge/atlassian-mcp/ATLASSIAN-MCP-CORRECTED-DIRECT-CLOUD.md)** ← **START HERE** — Direct cloud integration (no local setup)
- **[ATLASSIAN-MCP-CORRECTION-LOG.md](./knowledge/atlassian-mcp/ATLASSIAN-MCP-CORRECTION-LOG.md)** — Error analysis & lessons learned
- **[ATLASSIAN-MCP-DEPLOYMENT-PLAN.md](./knowledge/atlassian-mcp/ATLASSIAN-MCP-DEPLOYMENT-PLAN.md)** — Full 8-week deployment plan (Phase 1-5)
- **[ATLASSIAN-MCP-QUICKSTART.md](./knowledge/atlassian-mcp/ATLASSIAN-MCP-QUICKSTART.md)** — Developer quick reference
- **[ATLASSIAN-MCP-COMPARATIVE-ANALYSIS.md](./knowledge/atlassian-mcp/ATLASSIAN-MCP-COMPARATIVE-ANALYSIS.md)** — Why official Rovo MCP is the right choice

## 🎯 Quick Access

```bash
# Clone this repo
git clone git@github.com:MediaMarktSaturn/mms-agent-library.git

# View documentation
cat mms-agent-library/knowledge/ATLASSIAN-MCP-CORRECTED-DIRECT-CLOUD.md

# OR browse online
# https://github.com/MediaMarktSaturn/mms-agent-library/blob/main/knowledge/
```

## 📋 Contents

| Document | Purpose |
|----------|---------|
| `ATLASSIAN-MCP-CORRECTED-DIRECT-CLOUD.md` | How to integrate Atlassian MCP directly with pi (correct approach) |
| `ATLASSIAN-MCP-CORRECTION-LOG.md` | What I got wrong and why (anti-sycophancy log) |
| `ATLASSIAN-MCP-DEPLOYMENT-PLAN.md` | Comprehensive 8-week rollout plan with phases, risks, success criteria |
| `ATLASSIAN-MCP-QUICKSTART.md` | For developers: tools, workflows, examples |
| `ATLASSIAN-MCP-COMPARATIVE-ANALYSIS.md` | Comparison: Official Atlassian Rovo vs community alternatives |

## 🚀 Current Projects

- **Atlassian MCP Integration** (In Progress)
  - Status: Direct cloud integration configured in pi
  - Next: Phase 1 deployment (if approved)
  - Owner: Saúl (@sarony)

## 📝 Contributing

- All documents are Markdown
- Use same format as existing docs
- Link between related docs
- Update this README when adding new sections

## 📞 Support

- **Questions?** Ask Saúl (@sarony) or Tachikoma (@tachikoma)
- **Issues?** File in GitHub

---

**Repository**: MediaMarktSaturn/mms-agent-library  
**Last Updated**: 2026-07-15  
**Version**: 1.0
