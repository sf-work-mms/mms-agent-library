# Atlassian Rovo MCP Proxy — Executive Summary

**For**: AI Data Team, Platform Engineering Leadership  
**Date**: July 20, 2026  
**Status**: Ready for Review & Approval

---

## 🎯 The Ask

Deploy a **centralized Atlassian MCP proxy** in MMS infrastructure to enable AI agents and developers to securely access Jira Cloud, Confluence, and JSM through standardized MCP tools.

**Why?** Atlassian's official Rovo MCP endpoint is blocked by corporate whitelist policies. A proxy allows us to centralize access, implement audit logging, and support our AI automation roadmap.

---

## 📊 Business Impact

| Metric | Value |
|--------|-------|
| **Annual Benefit** | €250,000+ (productivity gains + automation) |
| **One-Time Investment** | €13,200 |
| **Annual Operating Cost** | €5,000 |
| **ROI (Conservative)** | 585% in Year 1 |
| **Payback Period** | 1.7 months |
| **Adoption Target** | 30% of platform engineers by Week 8 |

---

## 🏗️ Solution

```
Centralized Proxy (GCP VM)
  ↓ (OAuth 2.1, TLS 1.3)
Approved Whitelist ← IT Admin signs off
  ↓
Atlassian Cloud
  ↓
AI agents, Cursor, Claude, internal tools
```

**Key Features:**
- ✅ OAuth 2.1 authentication (Atlassian standard)
- ✅ 100% audit logging (who did what, when)
- ✅ Rate limiting + RBAC alignment
- ✅ Automatic token refresh (zero downtime)

---

## 📈 Use Cases Enabled

1. **Automated Issue Lifecycle**: Jira tickets auto-created from PRs, meeting notes, logs
2. **Smart Documentation**: Confluence pages auto-generated from code changes
3. **AI Triage**: JSM tickets auto-classified and routed
4. **Developer Velocity**: IDE tools (Cursor, Claude) query Jira/Confluence in real-time

---

## ⏱️ Timeline

| Phase | Duration | Outcome |
|-------|----------|---------|
| **1: Discovery** | Week 1-2 | Security review; whitelist request |
| **2: Deploy** | Week 3-4 | Proxy live; pre-auth testing |
| **3: Pilot** | Week 5-6 | 5-10 early adopters; feedback |
| **4: Harden** | Week 7 | Monitoring; documentation |
| **5: GA** | Week 8 | Rollout to all engineers |

**Total**: 8 weeks

---

## ✅ Success Criteria

- **Whitelist approval** from IT Admin (Week 2)
- **Security sign-off** from IT Security (Week 2)
- **Pilot adoption** ≥50% (5/10 users)
- **Error rate** <1%
- **Team adoption** ≥30% by Week 8

---

## 🚀 Next Steps

1. **This Week**: Review Business Case; align on budget
2. **Week 1**: Submit whitelist request to IT Admin + Atlassian Admin
3. **Week 2**: Security review complete
4. **Week 3**: Infrastructure deployment begins

---

## 📌 Key Decision

**Question for Leadership**: Approve €13.2K investment + 8-week implementation to unlock €250K+ annual benefit?

**Recommendation**: ✅ **YES** — Strong ROI, strategic alignment, enables AI roadmap.

---

## 📎 Supporting Documents

- **Full Business Case**: [BUSINESS-CASE-ATLASSIAN-MCP-PROXY.md](./BUSINESS-CASE-ATLASSIAN-MCP-PROXY.md)
- **Technical Architecture**: See Section 4 in full Business Case
- **Risk Analysis**: See Section 7 in full Business Case
- **Implementation Roadmap**: See Section 6 in full Business Case

---

**Contact**: Saúl Fernández (saul@mms.local)  
**Questions?** Schedule a 30-min walkthrough
