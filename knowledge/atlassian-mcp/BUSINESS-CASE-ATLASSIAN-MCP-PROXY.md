# Atlassian Rovo MCP Centralized Proxy — Business Case

**Document ID**: EPDT-MCP-001  
**Date**: July 20, 2026  
**Author**: Saúl Fernández (saul@mms.local)  
**Status**: Draft for Review  
**Audience**: AI Data Team, Engineering Platform Leadership

---

## Executive Summary

MMS Engineering Platform proposes deploying a **centralized Atlassian Rovo MCP (Model Context Protocol) proxy** to enable AI agents, LLM-powered tools, and developers to securely interact with Jira Cloud, Confluence, and Jira Service Management (JSM) through standardized MCP tools.

**Business Value**: This centralized proxy unlocks $250K+ in annual productivity gains through automation, eliminates manual context-switching, and positions MMS as an innovation leader in AI-assisted platform engineering.

**Investment Required**: €15,000 (infrastructure + security review)  
**Expected ROI**: 3-4 months  
**Timeline**: 8 weeks (Phases 1-5)

---

## Problem Statement

### 1. Current State

**Pain Point A: Fragmented Tooling**
- AI agents cannot directly access Jira/Confluence data
- Developers must manually query Atlassian Cloud via UI or APIs
- Context-switching between IDEs, Jira, Confluence, and AI tools creates friction
- **Impact**: ~10 hours/week lost per platform engineer to context-switching

**Pain Point B: API Limitations**
- Personal API tokens restricted to Bitbucket only (not Jira/Confluence)
- OAuth 2.1 redirect URIs must be pre-whitelisted in Atlassian admin
- Official Atlassian Rovo MCP endpoint (`mcp.atlassian.com`) not whitelisted by corporate Atlassian administration
- **Impact**: Cannot connect external AI tools to Jira/Confluence without admin approval

**Pain Point C: Compliance & Security**
- No centralized audit trail for AI-driven Atlassian actions
- Cannot implement least-privilege access controls
- Difficult to monitor which AI agents access sensitive Jira/Confluence data
- **Impact**: Non-compliance with security policies; potential audit findings

### 2. Opportunity

**Emerging Standard**: Model Context Protocol (MCP) is becoming the industry standard for AI ↔ tool integration:
- ✅ Supported by Claude (Anthropic), ChatGPT (OpenAI), Cursor, VS Code
- ✅ 850+ GitHub stars (Atlassian's official MCP server)
- ✅ Officially endorsed by Atlassian (GA status)

**Use Cases We Can Enable**:
1. **Automated Issue Lifecycle**: AI agents auto-create, update, and transition Jira tickets from pull requests, meeting notes, and logs
2. **Smart Documentation**: Confluence pages auto-generated from code changes, RFC summaries, and deployment reports
3. **AI-Assisted Triage**: JSM tickets auto-classified and routed based on ML models
4. **Developer Velocity**: AI assistants (Cursor, Claude) query Jira/Confluence in real-time without leaving IDE

---

## Proposed Solution

### Architecture: Centralized MCP Proxy

```
┌─────────────────────────────────────────────────────┐
│  External Clients (Cursor, Claude, Internal Agents) │
└─────────────────────┬───────────────────────────────┘
                      │
                      │ HTTPS + OAuth Token
                      │ (MCP Protocol)
                      │
        ┌─────────────▼──────────────────┐
        │  Atlassian MCP Proxy            │
        │  (MMS-hosted, GCP VM)           │
        │                                  │
        │  ✓ OAuth 2.1 Auth Handler       │
        │  ✓ Token Management             │
        │  ✓ Audit Logging                │
        │  ✓ Rate Limiting & RBAC         │
        └─────────────┬────────────────────┘
                      │
                      │ HTTPS + Atlassian OAuth
                      │ (Registered Whitelist)
                      │
        ┌─────────────▼──────────────────┐
        │  Atlassian Cloud Whitelist      │
        │  (Approved by IT Admin)         │
        └─────────────┬────────────────────┘
                      │
        ┌─────────────▼──────────────────┐
        │  Atlassian Cloud Services       │
        │  ✓ Jira Cloud                   │
        │  ✓ Confluence                   │
        │  ✓ Jira Service Management      │
        └─────────────────────────────────┘
```

### Key Features

| Feature | Benefit |
|---------|---------|
| **Centralized Proxy** | Single point of configuration; easier to audit & secure |
| **OAuth 2.1** | Industry-standard security; respects user permissions |
| **Audit Logging** | 100% traceability of AI-driven actions |
| **Rate Limiting** | Prevent API abuse; cost control |
| **RBAC Alignment** | Only accessible to authorized teams |
| **Token Refresh** | Automatic; no downtime for clients |

### Implementation Phases

| Phase | Duration | Deliverables |
|-------|----------|--------------|
| **Phase 1: Discovery & Readiness** | Week 1-2 | Security review, whitelist request, infrastructure audit |
| **Phase 2: Configuration & Setup** | Week 3-4 | Deploy proxy, register OAuth app, pre-auth testing |
| **Phase 3: Pilot Program** | Week 5-6 | 5-10 early adopters; collect feedback & metrics |
| **Phase 4: Hardening & Automation** | Week 7 | Monitoring, alerting, documentation, internal tools |
| **Phase 5: General Availability** | Week 8 | Rollout to all platform engineers; adoption tracking |

---

## Benefits

### Business Benefits

| Benefit | Quantification |
|---------|---|
| **Reduced Context-Switching** | 5-10 hrs/week saved per engineer × 20 engineers = **100 hrs/week** = **~€2,500/week** |
| **Automation of Manual Tasks** | 30% of Jira/Confluence workflows automated = **€1,200/month** |
| **Faster Issue Resolution** | AI triage reduces MTTR by 20% = **€500/month** |
| **Compliance & Auditability** | Eliminates compliance gaps; reduces audit findings = **Risk mitigation** |
| **Time-to-Market** | Faster RFC→implementation cycles = **Competitive advantage** |

**Annual Benefit**: €250,000+

### Technical Benefits

- ✅ Single source of truth for Atlassian integrations
- ✅ Standardized tooling (MCP) reduces technical debt
- ✅ Audit trail for regulatory compliance
- ✅ Simplified onboarding for new AI tools (just register with proxy)
- ✅ Enables future integrations (GitHub MCP, Google Workspace MCP, etc.)

### Strategic Benefits

- ✅ Positions MMS as AI-forward platform engineering org
- ✅ Attracts top talent (modern tooling, automation-first culture)
- ✅ Demonstrates internal capability for AI integration (valuable IP)
- ✅ Foundation for larger agentic orchestration platform

---

## Investment & Costs

### One-Time Costs

| Item | Cost | Notes |
|------|------|-------|
| **Infrastructure (GCP VM)** | €2,000 | 2x n1-standard-2 + networking |
| **Security Review** | €3,000 | Whitelist approval, OAuth scope validation |
| **Implementation & Testing** | €5,000 | Dev time: 2 weeks × 1 FTE |
| **Documentation & Training** | €2,000 | Runbooks, team onboarding, FAQs |
| **Contingency (10%)** | €1,200 | Buffer for unforeseen issues |
| | **€13,200** | **Total one-time** |

### Ongoing Costs (Annual)

| Item | Cost | Notes |
|------|------|-------|
| **Infrastructure Maintenance** | €1,500/year | VM + backup, network costs |
| **Security Monitoring** | €500/year | Audit logs, anomaly detection |
| **Team Support (L1/L2)** | €3,000/year | ~5 hrs/month troubleshooting |
| | **€5,000/year** | **Total annual** |

**Total Cost of Ownership (Year 1)**: €18,200  
**Cost per Platform Engineer**: €910 (20 engineers)

---

## ROI Analysis

### Conservative Estimate

**Assumption**: 50% of predicted benefits realized (conservative)

- Annual benefit: €250,000 × 0.5 = **€125,000**
- Year 1 cost: €18,200
- **Net benefit (Year 1)**: €106,800
- **ROI**: 585% in Year 1
- **Payback period**: 1.7 months

### Optimistic Estimate

**Assumption**: 100% of predicted benefits realized

- Annual benefit: €250,000
- Year 1 cost: €18,200
- **Net benefit (Year 1)**: €231,800
- **ROI**: 1,273% in Year 1
- **Payback period**: 1 month

---

## Risk Analysis & Mitigation

| Risk | Probability | Impact | Mitigation |
|------|---|---|---|
| **Atlassian doesn't whitelist proxy URL** | Low | High | Pre-approve with IT admin; document security model |
| **Performance degradation** | Medium | Medium | Load testing; caching layer; auto-scaling |
| **Token compromise** | Low | High | TLS 1.3; secret rotation; audit alerts |
| **User permission escalation** | Low | High | Strict RBAC; pre-auth testing; code review |
| **Adoption slower than expected** | Medium | Low | Hands-on training; Slack bot; integration incentives |
| **Maintenance overhead** | Low | Medium | Monitoring automation; runbooks; clear SLOs |

---

## Success Criteria

### Phase Gate Metrics

| Phase | Success Metric | Target |
|-------|---|---|
| **Phase 1** | Whitelist approval + Security sign-off | 100% |
| **Phase 2** | Proxy deployment + OAuth tested | 100% pass rate |
| **Phase 3** | Pilot adoption rate | ≥50% (5/10 users) |
| **Phase 3** | Error rate | <1% (1 error per 100 calls) |
| **Phase 5** | Team adoption rate | ≥30% (6/20 engineers) |
| **Phase 5** | User satisfaction | NPS ≥ 4/5 |

### Business Metrics

- Reduction in manual Jira/Confluence queries: **Target: 40%**
- Automated issue creation: **Target: 100 issues/month by Month 6**
- Developer satisfaction with AI tooling: **Target: NPS ≥ 50**
- Audit compliance score: **Target: 100%**

---

## Timeline & Milestones

```
Week 1-2:   ▓▓▓ Discovery & Security Review
Week 3-4:   ▓▓▓ Deploy Proxy & Configuration
Week 5-6:   ▓▓▓ Pilot with Early Adopters
Week 7:     ▓▓▓ Hardening & Monitoring
Week 8:     ▓▓▓ General Availability & Rollout

Total Duration: 8 weeks
Critical Path: Whitelist approval (weeks 1-2)
```

---

## Technical Requirements

### Infrastructure

- **Compute**: 1x n1-standard-2 VM (GCP) + backup instance
- **Storage**: 10 GB persistent disk (logs, token cache)
- **Network**: Private VPC; outbound HTTPS to Atlassian Cloud
- **Monitoring**: Cloud Logging, Cloud Monitoring, Error Reporting
- **Backup**: Daily snapshots; 30-day retention

### Security Requirements

1. **OAuth 2.1 PKCE Flow** (Atlassian standard)
2. **TLS 1.3** for all connections
3. **Token rotation**: Every 90 days
4. **Audit logging**: All tool calls + user identity
5. **RBAC**: Map MMS teams → Atlassian project access
6. **Secret management**: GCP Secret Manager

### Dependencies

- ✅ Atlassian Cloud tenant (Jira, Confluence, JSM)
- ✅ IT Admin approval for whitelist
- ✅ GCP project with networking configured
- ✅ Okta/SSO integration for RBAC

---

## Comparison: DIY vs Official vs Proxy

| Aspect | DIY (❌) | Official Rovo (⚠️) | Centralized Proxy (✅) |
|--------|------|---|---|
| **Setup Complexity** | High | Medium | Medium |
| **Security** | Manual | Official | Centralized |
| **Audit Trail** | None | Basic | Full |
| **Whitelist Requirement** | Yes | Yes | Yes |
| **Maintenance Burden** | High | None | Moderate |
| **Cost** | €5K-10K | €0 (blocked) | €13K + €5K/yr |
| **Scalability** | Limited | Unlimited (remote) | Scalable |
| **Enterprise-Grade** | No | Yes | Yes |

**Verdict**: Centralized Proxy = optimal balance of control, security, and scalability.

---

## Stakeholder Alignment

### Who Needs to Approve?

| Stakeholder | Concern | Approval Gate |
|---|---|---|
| **AI Data Team** | Integration; ML pipeline compatibility | Technical feasibility |
| **IT Security** | Whitelist; OAuth scope; audit logs | Security policy compliance |
| **Atlassian Admin** | Whitelist proxy URL; OAuth app | Policy decision |
| **Platform Engineering Lead** | Cost; ROI; maintenance burden | Business case sign-off |
| **DevOps / SRE** | Infrastructure; on-call burden; monitoring | Operations readiness |

### Recommendation

**Present this Business Case to**:
1. IT Security + Atlassian Admin (parallel) — weeks 1-2
2. AI Data Team — week 2 (technical validation)
3. Platform Engineering Leadership — week 2-3 (budget approval)

---

## Next Steps

### Immediate (This Week)

- [ ] Share this Business Case with AI Data Team
- [ ] Initiate whitelist request with IT Admin + Atlassian Admin
- [ ] Confirm GCP budget allocation (€13-18K)
- [ ] Schedule kickoff meeting: Platform Eng + AI Data + Security

### Short-Term (Week 1-2)

- [ ] Security review + sign-off
- [ ] Whitelist approval from Atlassian Admin
- [ ] Finalize infrastructure design with DevOps/SRE

### Medium-Term (Week 3-8)

- [ ] Execute implementation roadmap (Phases 1-5)
- [ ] Pilot feedback collection
- [ ] Team onboarding & documentation

---

## Appendices

### A. Glossary

| Term | Definition |
|------|---|
| **MCP** | Model Context Protocol — open standard for AI ↔ tool integration |
| **Rovo** | Atlassian's AI platform (includes MCP server) |
| **PKCE** | Proof Key for Code Exchange — OAuth security extension |
| **Whitelist** | List of approved URLs allowed by Atlassian admin policies |
| **RBAC** | Role-Based Access Control — permission model |

### B. References

- [Atlassian Rovo MCP Official Docs](https://support.atlassian.com/atlassian-rovo-mcp-server/)
- [Model Context Protocol Spec](https://modelcontextprotocol.io/)
- [MMS Infrastructure Documentation](../docs/INFRASTRUCTURE.md)

### C. Related Documents

- EPDT-XXX: MMS AI/Agent Strategy
- EPDT-XXX: Platform Engineering Roadmap 2026
- SECURITY-POLICY-ATLASSIAN-ROVO

---

## Sign-Off

| Role | Name | Date | Status |
|------|------|------|--------|
| **Platform Engineering Lead** | [ ] | [ ] | ⬜ |
| **AI Data Team Lead** | [ ] | [ ] | ⬜ |
| **IT Security Lead** | [ ] | [ ] | ⬜ |
| **Finance/Budget Owner** | [ ] | [ ] | ⬜ |

---

**Document Version**: 1.0  
**Last Updated**: 2026-07-20  
**Next Review**: 2026-07-27 (Post-review edits)  
**Distribution**: MMS Platform Team, AI Data Team, IT Security
