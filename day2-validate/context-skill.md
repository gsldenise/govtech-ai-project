# Context Skill · Passport Applicant Situational Reasoning

**Date:** 18 November 2026  
**Status:** Defines context for Week 4 validation testing

---

## Purpose

This skill enables the checker and document validator to distinguish applicant contexts—such as whether an applicant is a parent applying on behalf of a child, or an adult renewing their own passport—and surface only the relevant requirements and document flags for that context. It is tested by asking applicants to categorize their own situation through 2–3 branching questions, then observing whether they recognize mismatches between what the checker says they need and what they thought they needed before seeing the checker.

The riskiest assumption being tested: **Applicants who receive a situation-specific requirement summary and then see that some of their pre-submitted documents don't match the checklist will choose to go back and fix the documents (expected: ≥75%) rather than submit anyway (failure: ≥50%).**

Context determines:
1. Which documents are flagged as missing or invalid
2. What the applicant expected to need before the checker
3. Whether the applicant recognizes the gap and acts on it

---

## Sources

- [one-pager.md](../../one-pager.md): Hypothesis that situation-specific guidance reduces returns. Highest-pain segment: parents applying for child.
- [skill-file.md](../skill-file.md): Checker role—identify applicant situation through branching questions, return tailored one-screen summary.
- [build-log.md](./build-log.md): Week 4 validation gate requires ≥75% clarity on requirements (applicant describes what they'll prepare without looking back).
- [requirements-mapping.md](./prototype/requirements-mapping.md): Every requirement traces to ICA rule; scenarios include renewal-self, renewal-child.

---

## Rules

1. **Context is determined by applicant's own answers**, not by data they provide. Do not infer context from documents submitted.
2. **Context persists through submission flow.** Once an applicant says "I'm renewing my child's passport," all subsequent screens surface only child renewal requirements.
3. **Mismatch signals a contextual gap.** If applicant says they're applying for child, but submits only their own NRIC and no birth certificate, that's a context-specific miss—they didn't realize they needed the child's document.
4. **Context edge cases must escalate.** If applicant indicates "I'm applying for my child from overseas" or "My child's passport was lost," these fall outside standard scenarios and should flag to officer.
5. **Requirements must be scenario-specific, not generic.** Adult renewal shows different documents and fees than child renewal. Don't show both.

---

## Good Examples

### Example 1: Parent Applying for Child (Synthetic)

**Context identified:**
- Applicant type: Renewal
- For whom: Child (under 16)
- Age of child: 7 years old

**Requirements shown (scenario-specific):**
- Identity: Child birth certificate, digital passport photo (within 3 months), your Singpass login
- Supporting: Child's current passport (if any), your NRIC or valid passport, parental consent
- Fees: $70 application fee
- What to expect: Processing 1–2 weeks, child 6+ must collect in person

**Applicant's pre-checker assumption:** "I thought I just needed the child's photo and my own ID, like a normal photo ID renewal."

**Mismatch:** Applicant didn't realize they needed the child's birth certificate as a separate document.

**Validation test:** Applicant submits photo and NRIC, skips birth certificate. Validation flags missing birth certificate. Applicant sees "Go Back & Fix" option and realizes they need to gather the child's birth certificate before resubmitting. ✓ **Behavior test passes:** applicant chose to fix rather than submit incomplete.

---

### Example 2: Adult Renewal, Self (Synthetic)

**Context identified:**
- Applicant type: Renewal
- For whom: Myself
- Passport status: Expiring

**Requirements shown (scenario-specific):**
- Identity: Singpass login, NRIC number, digital passport photo (within 3 months)
- Supporting: Current passport number & expiry, name, DOB, sex, contact details, reason for renewal
- Fees: $70 application fee
- What to expect: Processing 1–2 weeks, passport valid 10 years, photo 400×514px JPG/PNG/HEIC/HEIF max 8MB

**Applicant's pre-checker assumption:** "I thought I'd just upload my current passport and a selfie."

**Mismatch:** Applicant expected a selfie would work; checker shows specific photo specs (400×514px, dated within 3 months, professional photo format).

**Validation test:** Applicant submits selfie. Validation flags "Photo outside 3-month window" and "Dimensions incorrect." Applicant sees specific issues and "Go Back & Fix" option. They schedule a photo appointment and resubmit. ✓ **Behavior test passes:** applicant acted on the feedback.

---

## Bad Example

### Example 3: Applicant Misses Context Shift (Synthetic)

**Context identified initially:**
- Applicant type: Renewal
- For whom: Myself

**Requirements shown:**
- Identity: Singpass, NRIC, digital photo (within 3 months)
- Supporting: Current passport number & expiry, contact details
- Fees: $70

**What went wrong:**
Applicant then realizing "Wait, I'm actually applying for my child, not myself" — but the checker does not re-trigger. Applicant submits under self-renewal context with child documents. System flags as mismatched.

**Why this is bad:** Context shift mid-flow breaks the assumption. The prototype doesn't handle "wait, I need to go back and change my scenario" — it assumes context is locked after the branching questions.

✗ **This is not a behavior test failure** (applicant didn't choose to submit incomplete). **This is a context architecture failure.** For validation, we ensure the learner cannot change context mid-flow without restarting.

---

## Owner

**Product team** — owns which scenarios are supported and whether edge cases escalate to officers. Frontline and product determine if context shift mid-flow is allowed or if restart is required.

---

## Review Trigger

- Ran Week 4 validation test with 5–8 applicants across 2–3 scenarios
- ≥75% of applicants correctly identified their own context (e.g., "parent applying for child" vs "adult renewal")
- ≥75% of applicants who saw a document mismatch chose "Go Back & Fix" rather than "Submit Anyway"
- Zero applicants tried to change context mid-flow (restart if needed)

**If triggered:** Advance to build phase (Week 5). If not triggered: Redesign branching questions or context prompt for clarity.

---

## Test Questions & Expected Answers

### Question 1: Does applicant recognize their own context after seeing the summary?

**Test:** Show applicant the requirement summary for their context. Ask: "Looking at this list, does it match what you expected to bring?"

**Expected answer:** Applicant identifies at least one gap. Example: "I didn't know I needed the birth certificate. I thought the photo and my ID were enough." OR "Yes, I expected all of this."

**Scoring:** ✓ If applicant can articulate the context (self vs child, renewal vs first-time). ✗ If applicant says "I don't know what any of this means."

---

### Question 2: When shown missing or invalid documents, does applicant understand why?

**Test:** Validation flags "Child birth certificate: Document was not submitted" and "Digital passport photo: Photo is outside 3-month window." Ask: "Why did the system flag these?"

**Expected answer:** Applicant restates the flag. Example: "I didn't upload the birth certificate at all." OR "My photo is too old."

**Scoring:** ✓ If applicant restates the issue in their own words. ✗ If applicant says "The system is broken" or "I don't know why it's rejecting this."

---

### Question 3: When presented with "Go Back & Fix" vs "Submit Anyway," which does applicant choose?

**Test:** Observe applicant's button selection after seeing flagged documents.

**Expected answer:** Applicant clicks "Go Back & Fix." If applicant clicks "Submit Anyway," ask why.

**Scoring:** ✓ If ≥75% of applicants (across all users) choose "Go Back & Fix." ✗ If ≥50% choose "Submit Anyway" — assumption is broken, checker doesn't change behavior.

**Recording:** Track outcome by context (parent-with-child, adult-renewal). If parent-with-child shows ≥75% fix rate but adult-renewal shows <50%, context clarity varies.

---

### Question 4: Can applicant describe what they'll prepare without looking back at the summary?

**Test:** After reading the summary, hide it and ask: "Walk me through what documents you'll gather before you submit. Don't look back at the list."

**Expected answer:** Applicant lists ≥2 of 3 core requirements for their context. Example (child renewal): "I need the child's birth certificate and my own ID, and then I need to get the child's photo taken."

**Scoring:** ✓ Pass if applicant accurately describes ≥2 of 3 core items. ✗ Fail if applicant lists generic items ("a photo and some paperwork") without specificity.

**This is the Week 4 clarity gate:** ≥75% of users must pass this question.

---

### Question 5: If context changes mid-flow, what happens?

**Test:** (Applicable only if applicant attempts to change scenario after branching questions are complete.) If applicant says "Wait, I need to go back and say I'm applying for my child instead," what does the system do?

**Expected answer:** System redirects to restart (goes back to step 1), or prevents context change mid-flow with a message: "To apply for a different scenario, please start over."

**Scoring:** ✓ If system has a clear path (restart or block). ✗ If applicant gets confused and mixed documents are submitted.

**Note:** This is not a user behavior question; it's an architecture check. For Week 4, we assume context is locked once branching questions are complete.

---

## Accuracy & Versioning

**Rules accurate as of:** 18 November 2026  
**Context scenarios tested:** Renewal-child, renewal-self  
**Edge cases flagged for escalation:** Overseas applicant, deceased parent, name change during passport validity  
**Next review:** After Week 4 validation (target: 25 November 2026)
