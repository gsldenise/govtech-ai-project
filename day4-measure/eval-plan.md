# Evaluation Plan: ICA Passport Photo Validator

**Scenario:** B · ICA passport applications  
**Date:** 20 November 2026  
**Purpose:** Define who owns the evaluation, when it runs, how to know it's working, and the roadmap for the first 30 days  
**Status:** All recommendations marked **Proposed** — stakeholder decides final approach

---

## Who Owns What

### Pass Bar Decision
**Owns:** Product Manager  
**Decides:** Which pass bar option (A/Strict, B/Balanced, C/Permissive, or D/A/B test) the agent must meet before Week 4 go-live  
**Currently selected:** Option B (Balanced) — deal-breakers 4/4 must PASS + 85% on other metrics  
**Approves:** Sponsor (escalation path if Option B not achievable)

### Test Cases (Create & Refresh)
**Owns:** Engineering Lead + Product Manager (joint)  
**Creates:** Initial 13 test cases (6 everyday, 3 tricky, 3 hand-off + 4 deal-breakers)  
**Refreshes quarterly:** Add new edge cases discovered in production, remove obsolete ones (e.g., "test case no longer applies after rule change")  
**Reviews:** Compliance & Data Governance Lead (ensures cases don't violate privacy, don't leak PII in test data)  
**Approves:** Product Manager

### Rubric Dimensions (Write & Maintain)
**Owns:** Product Manager + Engineering Lead (joint)  
**Writes:** 6 grading dimensions + calibration examples (worked examples Cases 1, 9, 13)  
**Maintains:** Update dimension definitions if validation rules change (e.g., "if confidence bands change, update Dimension 4 calibration")  
**Calibration:** Graders train against 3 worked examples; if >1 disagreement on any dimension, recalibrate

### Run the Evaluation
**Owns:** Engineering Lead or designated Lead Grader  
**Executes:** 
- For baseline (now): Score all 13 cases using rubric against CLAUDE.md version at time of test
- For weekly/monthly: Run 13 cases on current agent version, score, compare to baseline
- For A/B test: Run same 13 cases on both Auto and Confirm groups independently

**Timing:** 1–2 hours per evaluation run (13 cases × ~10 mins each)

### Review Weekly Sample (Spot Check)
**Owns:** Compliance & Data Governance Lead  
**Samples:** Every Friday, randomly select 2–3 of the 13 cases and verify grading logic was applied correctly  
**Flags:** If grader made systematic error (e.g., "always scored Clarity as 5/5 regardless of output"), notify Engineering Lead to retrain grader  
**Escalates:** To Sponsor if deal-breaker case was scored incorrectly (e.g., Case 10 marked PASS when should be FAIL)

### Pass/Fail Decision (Gate Keeper)
**Owns:** Product Manager (with Engineering Lead input)  
**Decides:** 
- "Agent passes baseline (all 13 cases score 13/13 and meet Option B)" → Ready for Week 4 test
- "Agent fails baseline (any deal-breaker fails)" → Escalate to Sponsor; hold deployment
- "Agent passes Option B but has pattern issue (e.g., Clarity always 2/5)" → Investigate root cause with EL; decide: fix or accept

**Approves:** Sponsor (if baseline not met or if pattern issue needs trade-off decision)

---

## When It Runs

### Before Go-Live (Week 4 Usability Test)
**Trigger:** Agent code frozen for Week 4 test  
**Timing:** Day before user test starts  
**Action:** Run all 13 cases against frozen agent version  
**Decision gate:** Pass Option B? YES → proceed to Week 4 test. NO → investigate + fix, rerun.

### Before Every Production Rule Change
**Trigger:** Any change to CLAUDE.md rules (e.g., confidence band update, new check added)  
**Timing:** After code change, before deployment  
**Action:** Run all 13 cases against updated agent  
**Decision gate:** Pass Option B? YES → safe to deploy. NO → fix or rollback.

### Weekly Operational Monitoring
**Trigger:** Every Monday 09:00  
**Timing:** 1 hour  
**Action:** Run 13 cases as smoke test; compare current results to baseline  
**Decision gate:** All 13 cases still passing at same score? YES → continue. NO or changed pattern → investigate (rule drift? agent bug?).

### When Policy Changes
**Trigger:** ICA policy change (e.g., "new photo brightness rule", "face-centering tolerance changes")  
**Timing:** Within 1 week of policy change  
**Action:** 
  1. Update CLAUDE.md with new rule
  2. Create new test case for the policy edge (e.g., "old brightness threshold now fails")
  3. Run all 14 cases (13 original + 1 new)
  4. Re-score + report

**Decision gate:** Pass Option B? YES → new policy live. NO → PM/EL agree on transition plan (grace period? grandfather old applicants?).

### A/B Test Comparison (Week 4)
**Trigger:** Week 4 usability test results in  
**Timing:** End of Week 4  
**Action:** Score Auto and Confirm groups separately on all 13 cases  
**Decision gate:** Which group has better deal-breaker pass rate + accuracy + applicant satisfaction? Pick winner for full deployment.

---

## How I'll Know It's Working

### The Goal
**Agent consistently validates passport photos to ICA spec, reduces false rejections (<10%), and applicants understand outcomes.**

### The Baseline (Established Now — 20 Nov 2026)
- **All 13 cases pass** against fixed CLAUDE.md (version commit 8177db6)
- **Option B met:** Deal-breakers 4/4 PASS, other metrics 100% PASS (exceeds 85% threshold)
- **No pattern issues:** All dimensions balanced (no Clarity always 2/5, no Escalation always 1/5)
- **Grading consensus:** All 3 calibration cases (1, 9, 13) scored exactly as rubric (0 disagreements)

### The Target (By End of Week 4 Usability Test)
- **Option B maintained:** At least 11/13 cases PASS; deal-breakers 4/4 PASS; ≥85% on other metrics
- **Engagement metric:** ≥25% of test users complete end-to-end validation
- **Clarity metric:** ≥75% of users can describe what photo checks mean (post-test survey)
- **No deal-breaker surprises:** Cases 4, 10, 12, 13 pass every run
- **Officer satisfaction:** >80% of FLAG cases have actionable officer notes (post-test survey)

### Review Date
**Formal decision point:** Friday, end of Week 4 (decision made Monday Week 5)  
**Decision:** Option B passed → Sponsor approves go-live OR Option B failed → hold, investigate, retrain agent.

### A Usage Metric to Watch
**Weekly accuracy trend:**
```
Week 1 (baseline): 13/13 PASS
Week 2 (monday): 13/13 PASS
Week 3 (monday): 13/13 PASS
Week 4 (usability test): [score both Auto and Confirm groups]
```
**Pattern to watch first:** Accuracy stays ≥13/13 or drifts down. If drift (e.g., "Week 2 = 12/13"), investigate immediately:
- Did CLAUDE.md rules change? (check git diff)
- Did grader calibration drift? (retrain on 3 worked examples)
- Did agent code drift? (check deployment logs)

### The Stop Line (Go/No-Go)
**Stop deployment if ANY of these occur:**
1. **Deal-breaker case fails** (Case 4, 10, 12, or 13 scores FAIL on its critical dimension)
2. **Option B threshold breached** (deal-breakers 4/4 still pass, but other metrics drop below 85%)
3. **Systematic grading error** (weekly spot check finds >1 dimension scored incorrectly on same case)
4. **Agent rule change without re-eval** (CLAUDE.md changed but 13 cases not re-run before deployment)

**Escalation:** Any stop line → Engineering Lead notifies Product Manager + Sponsor within 1 hour.

---

## First 30 Days

### Week 1 (20–26 Nov): Baseline Validation
**Goal:** Confirm all 13 cases still pass after recent rule fixes; lock in baseline scores.

**Monday 20 Nov:**
- ✅ Run all 13 cases against current CLAUDE.md (commit 8177db6)
- ✅ Verify all deal-breakers PASS
- ✅ Record baseline scores in eval-results.md archive section (add "Baseline — 20 Nov 2026")
- ✅ Confirm grader consensus on 3 calibration cases (1, 9, 13)

**Wed 22 Nov:**
- ✅ Grader training session: walk 2–3 graders through rubric + scorecard template
- ✅ Each grader independently scores 3 calibration cases
- ✅ Compare scores; if disagreement >1 dimension, recalibrate

**Fri 24 Nov:**
- ✅ Weekly spot check: sample 2–3 cases from baseline; verify grading logic
- ✅ Confirm no drift from baseline
- ✅ Report status: "Baseline locked. Ready for Week 4 test." → Sponsor

### Week 2 (27 Nov – 3 Dec): Preparation for Week 4 Test
**Goal:** Ensure agent + graders ready to evaluate usability test results.

**Monday 27 Nov:**
- ✅ Freeze CLAUDE.md (no rule changes until after Week 4 test evaluation)
- ✅ Run 13 cases again as "frozen snapshot" (verify no drift since baseline)
- ✅ Brief Product Manager: grading rubric, scorecard template, timeline for Week 4 results

**Wed 29 Nov – Fri 1 Dec:**
- ✅ Graders do dry-run scoring (practice on calibration cases again if uncertain)
- ✅ Engineering Lead prepares "A/B scorecard" template for Week 4 (separate sections for Auto group vs Confirm group)
- ✅ Compliance & Data Governance Lead spot-check: confirm test cases don't leak PII, don't violate privacy

**Fri 1 Dec:**
- ✅ Weekly spot check: sample 2–3 cases; confirm graders still aligned
- ✅ Status report: "Graders ready. Scorecards prepared. Awaiting Week 4 test data." → Sponsor

### Week 3 (4–10 Dec): Week 4 Test Runs (applies post-test, 10 Dec)
**Goal:** Collect A/B test data; score both groups independently.

**Monday 4 Dec – Fri 8 Dec:**
- 🔄 *Week 4 usability test running in parallel (not eval team's direct action)*
- ✅ Eval team: Prepare post-test analysis plan
  - Set up side-by-side A/B scorecard
  - Confirm Engineering Lead available to answer "what did the agent do?" questions during scoring

**Fri 8 Dec – Sat 9 Dec (evening):**
- ✅ Test results in; Eval team receives captured agent outputs (PASS/FLAG/FAIL, feedback, confidence, officer note) for all 13 cases × both groups
- ✅ Begin scoring: Auto group first (13 cases), then Confirm group (13 cases)
- ✅ Each grader scores one group independently (no cross-pollination)
- ✅ Record scores in "Eval Results — Week 4 A/B Test" section

**Sun 10 Dec (morning):**
- ✅ Compare scores; flag any systematic differences (e.g., "Auto group got Clarity 2/5 on average, Confirm got 4/5")
- ✅ Draft "A/B comparison report": which group passed Option B, which group had better accuracy, which group had clearer feedback
- ✅ Sponsor review: report + recommendation (Auto vs Confirm winner for full deployment)

### Week 4 (11–17 Dec): Decision & Next Steps
**Goal:** Sponsor decides go-live approach; Eval team documents lessons learned.

**Monday 11 Dec:**
- ✅ Sponsor briefing: A/B results + recommendation
- ✅ Sponsor decision: 
  - Option D (A/B test): Auto wins? → use Auto for full deployment
  - Option D (A/B test): Confirm wins? → use Confirm for full deployment
  - Option D (A/B test): Tie or unclear? → extend test, add new cases, or choose hybrid

**Wed 13 Dec:**
- ✅ Post-mortem meeting (PM, EL, OL, CO, Sponsor):
  - What surprised us? (e.g., "Clarity feedback was consistently vague across both groups")
  - What worked? (e.g., "Officer notes for FLAG cases were actionable")
  - What to fix before full deployment? (e.g., "Retrain on feedback clarity", "Update escalation criteria")

**Fri 15 Dec:**
- ✅ Weekly spot check: run 13 cases against current agent again; confirm no new drift
- ✅ Archive all Week 4 eval data (scorecards, A/B comparison, post-mortem notes) in eval-results.md
- ✅ Status report: "Week 4 test evaluated. [Winner] group recommended for deployment. Next step: [fix list] OR go-live prep." → Sponsor

**Mon 18 Dec – onward:**
- 🔄 Depending on Sponsor decision:
  - **Go-live path:** Eval team switches to "weekly operational monitoring" (see "When It Runs" section)
  - **Fix-and-retest path:** Eval team waits for fixes, reruns evaluation, repeats post-mortem

---

## Monthly Cadence (After Day 4)

Once live, evaluation runs:
- **Every Monday 09:00:** 13-case smoke test (1 hour)
- **Every Friday 14:00:** Weekly spot check on 2–3 random cases (30 mins)
- **1st Monday of month:** Governance review (full team, decision rights on any pattern issues)
- **When policy changes:** New test case added, all 13+ cases re-scored within 1 week

---

**Generated:** 20 November 2026  
**Scenario:** B  
**Status:** Evaluation plan ready for stakeholder sign-off; roles, timing, and success criteria defined
