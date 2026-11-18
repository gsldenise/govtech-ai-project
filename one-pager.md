# One Pager

Use case: Scenario B · ICA passport applications. Reduce return submissions through situation-specific completeness checks.

## Problem and user

**Who:** ~520,000 applicants per year submitting online; ICA officers processing returns.

**The gap:** Applicants submit incomplete or incorrect applications because they don't know what a complete application looks like for their specific situation. Generic guidance doesn't translate into what a particular applicant needs to do. Result: 17% return rate, each return adding 6 days to wait time.

**Highest-pain segment:** Parents applying for a child (34% of returns, but only 20% of applications). They don't understand they need both the child's documents and their own.

## Opportunity and evidence

**Goal:** Increase the proportion of passport applications approved on first submission so applicants get passports faster and ICA officers spend less time on returns.

**User evidence (synthetic):**
- "The form just said 'attach supporting documents'. I didn't know I needed both my own IC and my child's birth certificate."
- "They returned it after 10 days. The letter said 'incomplete documentation'. I had no idea what was missing."
- "I'd already booked the photos and taken half a day off work. When it came back I just felt like I'd wasted everyone's time."

**Metrics (synthetic):**
- Each return adds ~6 days to total wait time.
- Only 38% of applicants read guidance before starting.
- Parents applying for child account for 34% of returns, despite being 20% of applications.
- Top three return reasons: missing supporting document (41%), photo non-compliance (28%), incomplete or mismatched form fields (18%).

**Hypothesis:** Giving applicants a situation-specific completeness check at the point of submission — one that identifies their scenario and flags only relevant gaps — will reduce return rate, because most returns stem from applicants not knowing what their situation requires, not from unwillingness to comply.

## Experience and trust

**The moment:** When applicant uploads passport photos for submission.

**What applicant sees:** Upload photo, receive response on whether photo matches applicant and follows guidelines.

**Trust gaps and mitigations:**
- *Black box (AI says no, but why?)* → Errors must be in plain language so applicants can correct and resubmit.
- *Hallucination (good photo rejected)* → Robust user testing before deployment; clear confidence thresholds communicated.
- *Control (no override)* → User testing to validate whether applicants accept AI decisions or need human review option; trade-off: override may defeat purpose of automation.

**Confidence logic:**
- 90% or higher: submit application directly.
- 75–89%: submit with warning; officer flags for additional review.
- Below 75%: applicant must resubmit photo. System indicates what needs to change.

**Readiness level:** 2 of 3. Trigger Accuracy (Pass) and System Transparency (Pass). Invisible Integration not yet.

## Risks (market, technical, team)

**Market/timeline:**
- Vendor withdrawal deadline: 10 weeks. Hard stop. Whatever replaces the channel must be ready within that window.

**Technical:**
- Facial recognition and photo-guideline models must meet 90%+ accuracy to avoid rejecting compliant photos.
- SLA: AI must return validation results fast enough to keep applicant friction low.
- Data governance: Applications contain identity documents and sensitive personal data. Must comply with ICA's data handling requirements.

**Integration:**
- Must clearly separate what the AI checks (completeness, format) from what officers decide (eligibility, special circumstances).
- System must remain accurate as ICA rules change without requiring full rebuild each time.

## Build plan and handoff

**Validation phase:** Prototype a situational requirement checker (branching 3–4 questions) that returns a one-screen summary of what that applicant needs. Test with 5–8 users across scenarios (parent with child, adult renewal, lost passport, name change). Measure engagement and clarity.

**Handoff to build:** If validation shows ≥25% engagement and >75% clarity, move to full system integrating the checker into the submission flow with photo validation backend.

## Agent role and guardrails

**What the AI does:**
- Identifies applicant's situation through branching questions (no identity data collection).
- Returns tailored summary of requirements (identity documents, supporting docs, fees, what to expect).
- Validates passport photo format, size, and facial/guideline compliance.
- Flags applications 75–89% confidence to officer for additional review.

**What the AI does NOT do:**
- Collect or store name, NRIC, passport number, DOB.
- Make eligibility decisions or imply approval.
- Override officer judgment on special circumstances or edge cases.

**Every requirement listed must map to a specific ICA rule.** No synthesised or inferred requirements.

## Roles and operating model

[To be populated after validation phase]

## Strategy and roadmap

[To be populated after validation phase]

## Metrics plan and stop line

**Kill-switch signal:** Fewer than 25% of applicants who reach the submission page complete the checker in the first four weeks. If true, the engagement assumption is broken; no amount of checker quality compensates.

**Success signal:** Return rate falls from 17% to below 10% within three months of new channel going live, with the reduction concentrated in completeness-related returns (not officer-discretion returns). Parents applying for child show disproportionate improvement (≥50% drop in their return rate).

**Trust measure:** ≥90% of applications approved by AI without human intervention remain compliant when reviewed by officers.

## Evals and gates

[To be populated after validation phase]

## Decision: continue, pivot or stop

[To be determined after Day 2 validation]

## Open issues

- Whether applicants will engage with a pre-submission check when 62% currently skip written guidance.
- Whether the photo validation SLA is achievable within 10-week build window.
- How to handle edge cases (applicant overseas, deceased parent) without escalating every borderline case to officers.
- Whether allowing applicants to override AI photo rejection would improve trust or undermine the system's purpose.
