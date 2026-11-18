# Build Log · Passport Checker Prototype

**Date:** 18 November 2026  
**Scenario:** B · ICA passport applications · Reduce return submissions

---

## Stakeholder Review & Gate Decisions

### Round 1: Frontline Director (Operations)

**Their strongest objection:**  
"This checker doesn't solve the problem—it just pushes the problem into my team's court. You're asking applicants 3–4 questions before they submit. But 62% currently skip written guidance entirely. Your metric is 25% engagement—that's not solving, that's just accepting most won't use it. My officers still get the same 17% return rate, the same 6-day letters, the same backlog. You've just saved applicants 10 minutes of *not* reading."

**What would change their mind:**  
Evidence that engaged users (the 25% who complete the checker) see a measurably lower return rate for missing documents. Proof that the checker works for *those who use it*, not just a vanity metric.

**Resolution:**  
The checker will be **embedded into the application workflow**—not optional. Applicants cannot start the form without completing it. Additional validation checkpoints will run *during* the application process to catch errors before officers review. This converts 25% engagement from a floor to mandatory pre-submission filtration.

**Director's sign-off:** "If it's not optional and there are checkpoints during the form, you're preventing errors before they hit my desk. That's preventive, not reactive. I'm in."

---

### Round 2: Head of Product

**Their strongest objection:**  
"You're shipping a validation tool, not a product. This checker solves one transaction type, one moment, then the user leaves. No retention, no ecosystem, no scale. You're optimizing someone else's process on *my* product budget. Where's the next move? Is this a one-off or a platform?"

**What would change their mind:**  
1. Show the checker architecture scales to *N* application types without rebuilding each time (reusable pattern, not bespoke tool)
2. Prove **applicant reusability**: if a parent renews a child's passport again in 5 years, do they see a faster experience? Does the system remember their scenario?
3. Demonstrate that the 17%→10% return rate improvement frees officer capacity to reallocate to other services, creating indirect product velocity

**Resolution:**  
- **Reusable architecture**: The requirements-to-summary engine is modular. Other ICA application types (visa, citizenship, etc.) can plug in their own requirement mappings without re-engineering
- **Applicant memory**: On return visits, the system recognizes repeat applicants and pre-fills scenario (e.g., parent applying for child again → skip Q2)
- **Capacity reallocation**: Officers freed from return processing can handle more complex cases (overseas applications, special circumstances, name changes), expanding ICA's throughput
- **Day 3/4 deliverable**: Architecture diagram showing how scenarios, requirements, and validations are decoupled and reusable

**Product's sign-off:** "That's product thinking. Not a leak fix, a pump. I'm in, pending you show me the pattern in Day 3/4 output."

---

### Round 3: Chief Executive Sponsor

**Their strongest objection:**  
"You're asking me to replace a critical vendor channel in 10 weeks on a hypothesis that hasn't been tested. The risk isn't the checker—it's the timing. One bad week and we miss the deadline, applications grind to a halt, my minister gets called into Parliament."

**What would change their mind:**  
1. **Fallback plan**: Checker is modular—can be toggled off without breaking the application channel. Core form works standalone.
2. **Technical SLA**: Photo validation completes in <5 seconds, 99.9% uptime
3. **Risk gate at Week 4 of validation**: If engagement <25% OR clarity <75%, pause build, run second design sprint, delay launch to 12 weeks (vendor grants grace period)

**Resolution:**

#### Week 4 Validation Gate

**Engagement:** ≥25% of test users complete the checker end-to-end  
**Clarity:** ≥75% of users can *describe what they need to do* without referencing the summary

**Clarity Test Protocol:**
- After seeing the summary, ask: "Walk me through what you'll prepare before submitting, without looking back at the screen"
- Pass if user accurately describes ≥2 of 3 core requirements for their scenario and explains why each is needed
- Test with 5–8 users (parent-with-child, adult renewal, one special case)
- Target: ≥6 out of 8 pass

**Gate decision:**
- **If ≥25% engagement AND ≥75% clarity:** Proceed to build (Week 5–9)
- **If <25% engagement OR <75% clarity:** Pause build, run one-week design sprint, re-test Week 5. If still miss gates, pivot to different UX pattern (guided form instead of card summary). Launch pushed to Week 12 with vendor grace period.

#### Week 9 Photo Validation SLA Gate

**Requirement:** Photo validation completes in <5 seconds, 99.9% uptime

**If SLA is met:** Proceed to production launch Week 10  
**If SLA is missed:** Tech lead chooses one path by Week 9:
- Optimize model to 5 seconds
- Fall back to slower but reliable synchronous server-side check
- Reduce scope to photo format validation only (not facial compliance)

**If none of those work by Week 9:**
- Launch without photo AI (requirements checker only)
- Requirements checker still addresses 41% of returns (missing documents)
- Photo validation becomes Phase 2, post-launch, once SLA is proven in staging

#### Requirement Change Management

**Owner:** Product team  
Monitors ICA announcements weekly. If ICA changes a requirement mid-build, product owns the decision: patch the checker mid-build or pause and re-scope. Not engineering's problem to solve.

**CEO's sign-off:** "Modularity means we're not all-in on the checker. It's insurance, not the whole bet. Week 4 gate and Week 9 gate are your release checkpoints. Hit both, we ship Week 10. Miss either, we pause and re-plan. Clear."

---

## Commitments & Timeline

| Milestone | Owners | Gate | Criteria |
|---|---|---|---|
| **Week 1–2: Test prep** | Product, Design, Frontline | None | 5–8 users recruited, test script final, prototype ready |
| **Week 3–4: Validation** | Product, Design, User Research | Week 4 gate | ≥25% engagement, ≥75% clarity on requirements description |
| **Week 5–9: Build & integration** | Engineering, Product, Frontline | Week 9 gate | Photo SLA <5 sec, 99.9% uptime; modular architecture proven |
| **Week 10: Production launch** | Engineering, Operations | Release | Core application channel + checker live, fallback tested |

---

## Architecture Principles (For Day 3/4 Design)

1. **Modularity**: Checker can be toggled off without breaking application channel
2. **Core form works standalone**: If checker fails, application processing continues
3. **Reusability**: Requirements-to-summary engine decoupled from scenario logic (scenarios = data, not code)
4. **Stateful UX**: System remembers applicant profile on repeat visits (parent with child, adult renewal, etc.)
5. **Non-blocking gates**: Photo validation is async; applicant receives status email, doesn't block submission

---

## Known Risks & Mitigations

| Risk | Mitigation | Owner | Gate |
|---|---|---|---|
| Photo validation SLA unachievable | Reduce scope to format validation; async fallback | Engineering | Week 9 |
| User engagement <25% | Redesign copy/layout; one-week sprint; retest | Product, Design | Week 4 |
| Clarity <75% | Pivot to guided form; delay launch to Week 12 | Product, Design | Week 4 |
| ICA requirement change mid-build | Product owns decision; patch or pause-and-rescope | Product | Ongoing |
| Core form breaks due to checker integration | Modular design; checker can toggle off instantly | Engineering | Week 5 pre-flight |

---

## Success Definitions (Post-Launch)

**Kill-switch signal (stop the project):**
- Fewer than 25% of applicants who reach the submission page complete the checker in the first four weeks

**Success signal (continue investment):**
- Return rate falls from 17% to below 10% within three months of launch
- Reduction concentrated in completeness-related returns (not officer-discretion returns)
- Parents applying for child show ≥50% drop in their return rate
- ≥90% of applications approved by AI photo validation remain compliant when reviewed by officers

---

## Decisions Log

| Date | Decision | By | Status |
|---|---|---|---|
| 18 Nov | Frontline Director approves embedded checker + workflow checkpoints | Operations Lead | ✅ Approved |
| 18 Nov | Head of Product approves reusable architecture approach | Product Lead | ✅ Approved, pending Day 3/4 architecture doc |
| 18 Nov | CEO approves 10-week timeline with Week 4 & Week 9 gates | Executive Sponsor | ✅ Approved |
| 18 Nov | Photo validation SLA set to <5 sec, non-negotiable | Executive Sponsor | ✅ Non-negotiable |
| 18 Nov | Product team owns ICA requirement changes | Executive Sponsor | ✅ Assigned |

---

## Next Steps

- [ ] **Day 2 (Week 3–4):** Run validation test with 5–8 users; measure engagement & clarity
- [ ] **Day 3 (Week 5–7):** Architecture design; reusable scenario engine; stateful applicant profile
- [ ] **Day 4 (Week 8–10):** Build, integrate, test; photo SLA validation; production readiness
- [ ] **Week 9:** Go/no-go decision on SLA; escalate to CEO if missed
- [ ] **Week 10:** Launch or execute fallback plan

---

## Document Ownership

**Created by:** Product team (with input from Frontline Director, Head of Product, Executive Sponsor)  
**Last updated:** 18 November 2026  
**Next review:** After Week 4 validation gate
