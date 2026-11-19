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

## Incident Response (08:00 Monday Scenario: Agent Sends Wrong Answer)

### Timeline & First Responders

**08:00 — Alert triggered**
- **First responder:** On-duty EL (or duty engineer)
- **Detection:** Automated alert (accuracy drop, false rejection spike) OR manual report from OL/PM
- **Initial action:** Confirm incident is real (not false alarm) — take 5 mins max

**08:05 — Severity Assessment & Go/No-Go Decision**

| Severity | Scenario | First Responder Acts | Authority to Switch Off |
|----------|----------|---------------------|----------------------|
| P1 (Critical) | Agent sending wrong answer to ALL users; system trust compromised; >100 applicants affected within last hour | EL: DISABLE AGENT IMMEDIATELY (kill switch) without waiting for approval | EL (immediate authority) |
| P2 (High) | Agent sending wrong answer to <100 users; accuracy <80%; false rejection spike; affecting <1% of daily volume | EL: Escalate to Sponsor within 15 mins; Sponsor decides disable/continue-with-monitoring | Sponsor (after EL escalation) |
| P3 (Medium) | Single user complaint; isolated wrong answer; accuracy still >90%; <1 applicant affected | OL/PM: Log, investigate; notify Sponsor within 1 hour | Not disabled; monitoring escalated |
| P4 (Low) | Suspected issue but unconfirmed; no applicant impact yet | EL: Investigate; no escalation needed if confirmed false alarm | Not disabled |

**Action at 08:05 (assuming P1 — agent sends wrong answer to multiple users):**

**EL: Switch off agent immediately**
- Set `validation_enabled = false` in config
- All uploads now: "Validation temporarily unavailable. Please contact support."
- Log timestamp, reason, who disabled it

**08:05 — Notify Incident War Room**

**First message (EL → Sponsor, PM, OL, CO):**
- What: Agent validation sending wrong answers
- When: Started 08:00, discovered 08:05
- Who: ~[N] applicants affected, ~[N] photos incorrectly rejected/passed
- Action taken: Agent disabled at 08:05
- Next: War room call at 08:15

**Sponsor receives first notification within 5 mins of switch-off**

**08:15 — Incident War Room Convenes**

**Attendees:** Sponsor, EL, PM, OL, CO (all required)

**Agenda (max 30 mins):**
1. What happened? (EL presents logs, accuracy metrics)
2. Root cause? (EL: bug, data corruption, model drift, config error?)
3. Scope of damage? (PM/OL: how many affected, what happened to them?)
4. Compliance check? (CO: any PII exposed, breach?)
5. Remediation plan (EL)
6. Comms plan (PM) — what to tell affected applicants?
7. Sponsor decision: Rollback to previous version? Or fix forward?

**08:45 — Sponsor Communicates to ICA Leadership**

**PM drafts (Sponsor approves) message to ICA Sponsor's boss:**
- Incident identified and contained (agent disabled)
- Scope: [N] applicants, [status] of impact
- Root cause: [preliminary]
- Timeline to fix/resume: [ETA]
- No PII leaked: [CO confirmation]
- Remediation plan: [brief]

**09:00+ — Fix Forward or Rollback**

**If fixable within 2 hours (P2 decision):**
- EL: Implement fix, test staging, get approval
- Sponsor: Approve re-enable
- OL: Monitor; be ready to accept FLAG queue surge

**If not fixable within 2 hours (P1 persistence):**
- EL: Rollback to last known-good version
- Sponsor: Re-enable with monitoring
- PM: Communicate delay to ICA stakeholders

**09:30+ — Post-Incident**
- EL: Enable agent with increased monitoring (alert on first accuracy drop)
- PM: Monitor for second wave of complaints
- Sponsor: Brief ICA leadership on resolution + next steps

---

### Incident Contact Tree (for 08:00 Monday)

**On Duty 08:00 Monday:** 
- Primary: EL (has kill switch authority)
- Secondary: PM (monitors metrics)
- Tertiary: OL (manages officer queue impact)

**Escalation Chain:**
1. **First responder (EL):** Detects issue → disables agent (if P1) OR escalates to Sponsor (if P2+)
2. **Sponsor:** Makes disable/continue decision (P2+); approves comms; briefs ICA leadership
3. **PM:** Drafts applicant comms; manages reputation impact
4. **OL:** Manages officer workflow disruption (FLAG queue backup, manual review surge)
5. **CO:** Checks for PII/compliance breach; audit log review

**Who tells the business owner (Sponsor):**
- **First notification (08:05):** EL (factual: what, when, action taken)
- **Follow-up (08:15):** EL (technical details), PM (business impact), CO (compliance check)
- **External comms (~08:45):** PM (drafted), Sponsor (approved + sent to ICA leadership)

---

### Missing Details Now Filled In

✅ **First responder:** On-duty EL  
✅ **Switch-off authority:** EL (immediate for P1); Sponsor (for P2+)  
✅ **Severity levels:** P1 (disable immediately), P2 (escalate within 15 mins), P3 (log, investigate), P4 (false alarm)  
✅ **Timeline:** Alert 08:00 → assess 08:05 → war room 08:15 → comms 08:45 → fix 09:00+  
✅ **Notification sequence:** EL → Sponsor → PM (applicant comms) → ICA leadership  
✅ **Containment:** Disable agent, set "temporarily unavailable" message  
✅ **On-duty contact tree:** EL primary, PM secondary, OL tertiary

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
