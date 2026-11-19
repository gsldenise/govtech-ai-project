# Operating Model: ICA Passport Photo Validator

**Scenario:** B · ICA passport applications  
**Agent:** Photo Validator (CLAUDE.md)  
**Date:** 19 November 2026  
**Governance focus:** Clear decision rights, escalation paths, accountability

---

## Roles

**Product Manager (PM):** Represents ICA stakeholders, sets requirements, owns the business case. Approves scope changes, success metrics, go/no-go decisions. Owns engagement metrics (% completing validation), clarity post-test, user feedback synthesis, iteration prioritization.

**Engineering Lead (EL):** Leads SWE team. Owns technical implementation, system architecture, performance SLA, security posture. Makes technical trade-offs within approved scope.

**ICA Sponsor (Sponsor):** Represents ICA stakeholders; ICA leadership accountable to Ministry/leadership. Approves vendor partnerships, production deployment, budget escalations, policy changes.

**Compliance & Data Governance Lead (CO):** Ensures data governance, privacy rules, audit trails, PII handling. Reviews logging, retention policies. Custodian of data retention, deletion, audit logs. Reports compliance status. Manages 30-day retention enforcer. Flags breaches.

**Officer Lead (OL):** Manages ICA photo validation officers. Owns officer workflow (FLAG queue, approval SLA). Escalates officer capacity issues, feedback on system accuracy.

---

## Decision Rights Table

| Decision | Harness Part | Who Decides | Who is Consulted | Who is Told |
|----------|--------------|-------------|-----------------|-------------|
| **SCOPE** | What the agent validates (10-point checklist), what it doesn't (eligibility, authenticity) | PM | EL, Sponsor, CO | PM, OL, CO |
| | Feature adds/removals (e.g., add "passport taken <3 months" check) | PM | EL, CO | PM, OL, all stakeholders |
| | Edge case escalation policy (FLAG vs. FAIL vs. reject) | PM | OL, EL, PM | Sponsor, CO, PM |
| | **What triggers: "Out of scope for this agent"** (deepfakes, authenticity, eligibility) | PM | EL, CO, OL | Sponsor |
| **PERMISSIONS** | Which checks are deterministic vs. simulated (frozen until real ML) | EL | PM, CO | Sponsor, PM, OL |
| | ML pass rates (70% expression, 80% clarity, etc.) | EL | PM, PM | Sponsor, OL |
| | Confidence bands (PASS ≥90%, FLAG 75–89%, FAIL <75%) | PM | EL, PM, OL | Sponsor, PM |
| | What PII is rejected (name, NRIC, DOB, passport number) | CO | PM, EL | Sponsor, CO, EL |
| | What gets logged vs. what is anonymized | CO | EL, CO | Sponsor, CO, all |
| | Upload blocking rules (failed photo → must resubmit) | PM | EL, OL | PM, OL |
| | **Confidence threshold changes (e.g., PASS moves to 95%)** | PM | EL, PM, OL | Sponsor, all |
| **VENDOR & MODEL** | Real ML model selection (when simulated → real) | EL | PM, Sponsor, CO | Sponsor, PM |
| | Face detection library (face.js, TensorFlow.js, etc.) | EL | PM | Sponsor |
| | Cloud logging service (GCP Logs, Datadog, etc.) | EL | CO, CO | Sponsor, CO |
| | Whether to do client-side or server-side validation | EL | PM | Sponsor, PM |
| | Browser/platform support matrix (IE11? iOS Safari?) | EL | PM | Sponsor |
| | **Third-party API costs/contracts** | Sponsor | EL, PM | Finance, PM |
| **GO-LIVE** | Release readiness (gates: engagement ≥25%, clarity ≥75%) | PM | PM, EL | Sponsor, all |
| | Production deployment approval | Sponsor | PM, EL, CO, CO | All stakeholders |
| | Cutover from prototype to live system | Sponsor | EL, OL | OL, all support teams |
| | Applicant communication (how to explain changes) | PM | PM, OL | Sponsor |
| | Officer training (how to review FLAG cases) | OL | PM, PM | Sponsor, officers |
| | **Go-live rollback decision (kill switch if problems)** | Sponsor | EL, PM, OL | All |
| **INCIDENT** | System performance degradation (validation >5sec) | EL | PM, OL | Sponsor, OL, PM |
| | Photo validation accuracy drops below 90% | PM | EL, OL, CO | Sponsor, PM |
| | PII accidentally logged | CO | CO, EL | Sponsor, CO, Audit |
| | Officer FLAG queue overflows (SLA missed) | OL | PM, PM | Sponsor, PM |
| | False rejection rate spikes (applicant complaints) | PM | PM, OL, EL | Sponsor, PM |
| | Suspected deepfakes detected in validation stream | OL | CO, EL | Sponsor, CO, PM |
| | **System outage or major bug** | Sponsor | EL, PM | All |
| | Incident communication to ICA leadership | Sponsor | PM, CO | All stakeholders |
| **RETIRE** | When to sunset the agent (e.g., real ML makes it obsolete) | Sponsor | PM, EL | All stakeholders |
| | Data deletion timeline (30-day retention enforcement) | CO | CO, EL | Sponsor, Audit |
| | Cutover plan from agent to replacement system | Sponsor | EL, PM | All |
| | Final validation accuracy audit before shutdown | PM | EL, CO | Sponsor, Audit |
| | Officer workflow transition (e.g., back to manual review) | OL | PM | Sponsor, officers |
| | **Lessons learned / post-mortem** | PM | EL, PM, OL, CO | Sponsor |

---

## Escalation Paths

**When does a decision escalate?**

- **Technical blocker:** EL → Sponsor (e.g., ML model unavailable, performance target unachievable)
- **Scope creep:** PM → Sponsor (e.g., "add eligibility check" — beyond agent's role)
- **Policy conflict:** CO → Sponsor (e.g., data retention requirement conflicts with operational need)
- **Capacity crisis:** OL → Sponsor (e.g., FLAG queue overwhelmed, officers can't keep pace)
- **Accuracy crisis:** PM → Sponsor (e.g., false rejection rate >20%, applicants losing trust)
- **Compliance breach:** CO → Sponsor (e.g., PII accidentally logged)

**Sponsor resolves ties.** If PM and EL disagree on feasibility vs. requirements, Sponsor decides.

---

## Communication Cadence

| Forum | Frequency | Attendees | Topics |
|-------|-----------|-----------|--------|
| Weekly Standup | Every Monday | EL, PM, OL, PM | Progress, blockers, metrics on track? |
| Metrics Review | Every Friday | PM, PM, OL | Engagement %, clarity post-test, FLAG queue depth, accuracy trends |
| Incident War Room | On-demand | All | Immediate escalation if SLA missed, PII breach, outage |
| Monthly Governance | 1st of month | PM, EL, CO, CO, OL, Sponsor | Compliance audit, data retention enforcement, lessons learned |
| Quarterly Business Review | End of quarter | Sponsor, PM, EL, PM | Go/no-go decision for next phase, budget, roadmap |

---

## Success Criteria (Who Owns What)

| Metric | Target | Owner | Failure Response |
|--------|--------|-------|------------------|
| **Engagement (Week 4 test)** | ≥25% of users complete end-to-end | PM | If <25%: iterate on UX, retest; escalate to Sponsor if retest <25% |
| **Clarity (Week 4 test)** | ≥75% can describe requirements post-test | PM | If <75%: iterate messaging, retest; escalate to Sponsor if retest <75% |
| **Photo validation accuracy** | ≥90% of agent decisions approved by officers | EL, OL | If <90%: retrain/adjust rules; escalate to Sponsor if unfixable |
| **FALSE REJECTION rate** | <10% (applicant complaints) | PM | If ≥10%: investigate causes, adjust confidence bands; escalate to PM |
| **Officer FLAG queue SLA** | 95% resolved within 24 hours | OL | If missed: escalate capacity issue to Sponsor |
| **System uptime** | 99% availability | EL | If <99%: post-mortem + incident plan; escalate to Sponsor |
| **Data compliance** | 0 PII breaches, 30-day retention enforced | CO, CO | If breach: incident escalation to Sponsor + audit |

---

## Change Management

**Minor changes (no escalation needed):**
- PM-approved wording tweaks on feedback messages → EL implements
- Logging dashboard metric additions → CO approves, EL implements
- Officer SLA adjustments (24h → 48h) → OL proposes, PM approves

**Major changes (escalate to Sponsor):**
- Confidence band changes (75–89% becomes 70–85%) → PM proposes, Sponsor decides
- New validation check added (e.g., "photo taken <3 months") → PM proposes, Sponsor decides
- ML model replacement (simulated → real) → EL proposes, Sponsor decides
- Feature sunset (retire agent) → Sponsor decides

**Security/compliance changes (CO reviews):**
- PII handling changes (add/remove fields) → CO approves before implementation
- Data retention policy changes → CO + CO approve

---

## Handoff Milestones

| Milestone | Owner | Go/No-Go Criteria | Next Owner |
|-----------|-------|------------------|------------|
| **Week 4: Usability Test Complete** | PM | Engagement ≥25% AND clarity ≥75% | PM |
| **Week 5: Gate 1 Decision** | PM | Test results meet criteria + ICA sign-off | EL |
| **Week 6: Production Build Start** | EL | Spec approved, 20 implementation gaps resolved | EL |
| **Week 8: Staging Ready** | EL | Testing complete, accuracy ≥90%, SLA met | CO |
| **Week 9: Production Deploy** | PM, Sponsor | Final sign-off, officer training done, rollback plan ready | OL, EL |
| **Week 10: Live Validation** | OL, PM | Monitoring live metrics, incident response active | All |

---

**Generated:** 19 November 2026  
**Scenario:** B  
**Status:** Ready for stakeholder sign-off on decision rights and roles
