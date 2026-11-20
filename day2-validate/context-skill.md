# Context Skill · Passport Applicant Situational Reasoning

**Date:** 20 November 2026  
**Status:** Defines applicant task for Week 4 usability testing

---

## Purpose

This skill tests whether applicants can recognize their own passport renewal situation through self-guided branching questions, understand the situation-specific requirements, gather the right documents, and successfully submit with confidence. The applicant task is to answer 2–3 screening questions ("Are you renewing your own passport?" / "Are you renewing for a child?"), review the tailored requirement summary, upload documents that match, and recognize when documents don't fit the requirements.

The riskiest assumption being tested: **Applicants who answer context questions correctly, review the requirement summary, and then upload documents will recognize mismatches and choose to go back and fix them (expected: ≥75%) rather than submit anyway (failure: ≥50%).**

Applicant clarity determines:
1. Whether applicant correctly identifies their own situation (renewal-self vs renewal-child)
2. Whether applicant understands what documents their situation requires
3. Whether applicant recognizes when uploaded documents don't match and acts to fix them

---

## Sources

- [one-pager.md](../../one-pager.md): Hypothesis that situation-specific guidance reduces returns. Highest-pain segment: parents applying for child.
- [full-flow.html](./prototype/full-flow.html): Applicant self-serve journey—answer branching questions, review requirements, upload documents, get validation feedback.
- [build-log.md](./build-log.md): Week 4 validation gate requires ≥75% clarity on requirements (applicant describes what they'll prepare without looking back).
- [requirements-mapping.md](./prototype/requirements-mapping.md): Every requirement traces to ICA rule; scenarios include renewal-self, renewal-child.

---

## Rules

1. **Applicant context comes from their own answers**, not from documents. Do not infer context from what they upload.
2. **Context persists through the upload flow.** Once an applicant says "I'm renewing my child's passport," all screens show only child renewal requirements.
3. **Document mismatch signals unclear requirements.** If applicant says they're applying for child but uploads only their own NRIC and no birth certificate, the requirement wasn't clear enough—they didn't realize they needed the child's document.
4. **If applicant is unsure of their own situation, escalate.** Applicant says "I don't know if I'm renewing for my child or myself" → flag for officer guidance (not an applicant-side fix).
5. **Requirements must be scenario-specific, not generic.** Adult renewal shows different documents and fees than child renewal. Don't make applicant choose.

---

## Good Examples

### Example 1: Parent Applying for Child (Synthetic)

**Applicant's situation (from branching questions):**
- Type: Renewal
- For whom: Child (under 16)
- Child's age: 7 years old

**Requirements applicant sees (scenario-specific):**
- Identity: Child birth certificate, digital passport photo (within 3 months), your Singpass login
- Supporting: Child's current passport (if any), your NRIC or valid passport, parental consent
- Fees: $70 application fee
- What to expect: Processing 1–2 weeks, child 6+ must collect in person

**Applicant's pre-checklist assumption:** "I thought I just needed the child's photo and my own ID, like a normal photo ID renewal."

**Applicant recognizes the gap:** After reviewing requirements, applicant realizes "Oh! I need the child's birth certificate as a separate document. I didn't have that on my list."

**Applicant acts on the gap:** Applicant uploads photo and NRIC, but skips birth certificate. Validation flags missing birth certificate. Applicant sees "Go Back & Fix" option and thinks: "Right, I need to get the child's birth certificate first." ✓ **Behavior test passes:** applicant corrects the submission.

---

### Example 2: Adult Renewal, Self (Synthetic)

**Applicant's situation (from branching questions):**
- Type: Renewal
- For whom: Myself
- Passport status: Expiring

**Requirements applicant sees (scenario-specific):**
- Identity: Singpass login, NRIC number, digital passport photo (within 3 months)
- Supporting: Current passport number & expiry, name, DOB, sex, contact details, reason for renewal
- Fees: $70 application fee
- What to expect: Processing 1–2 weeks, passport valid 10 years, photo 400×514px JPG/PNG/HEIC/HEIF max 8MB

**Applicant's pre-checklist assumption:** "I thought I'd just upload my current passport and a selfie."

**Applicant recognizes the gap:** After reviewing requirements, applicant sees "Wait, I need a very specific photo size (400×514px)? A selfie won't work. And it has to be recent (within 3 months)."

**Applicant acts on the gap:** Applicant uploads a selfie. Validation flags "Photo outside 3-month window" and "Dimensions incorrect (375×420 vs required 400×514)." Applicant sees specific issues and thinks: "I need to schedule a professional photo appointment." ✓ **Behavior test passes:** applicant understood the requirement and is fixing it.

---

## Bad Example

### Example 3: Applicant Doesn't Recognize Their Situation (Synthetic)

**Applicant answers context questions:**
- Type: Renewal
- For whom: Myself

**Requirements shown:**
- Identity: Singpass, NRIC, digital photo (within 3 months)
- Supporting: Current passport number & expiry, contact details
- Fees: $70

**What went wrong:**
Applicant then realizes "Wait, I'm actually applying for my child, not myself"—but the system is already locked in the self-renewal flow. Applicant uploads child documents anyway. System flags as mismatched (child docs don't fit self-renewal context).

**Why this is bad:** Context confusion is not an applicant skill to test—it's a system design issue. For Week 4, we assume context questions are clear enough that applicants don't second-guess themselves.

✗ **This is not a behavior test failure** (applicant didn't choose to submit incomplete). **This is a question clarity failure.** If >5% of applicants get their context wrong, we redesign the branching questions.

---

## Owner

**Product Manager** — owns which scenarios are supported and whether unclear situations get escalated to officers. Responsible for ensuring branching questions are clear enough that applicants identify their own situation correctly on first try.

---

## Review Trigger

- Ran Week 4 usability test with 5–8 applicants across 2–3 scenarios
- ≥75% of applicants correctly identified their own context (self vs child renewal)
- ≥75% of applicants who saw a document mismatch chose "Go Back & Fix" rather than "Submit Anyway"
- ≥75% of applicants can describe required documents after reviewing the summary (clarity gate)
- Zero applicants got their context wrong mid-flow and submitted mismatched documents

**If triggered:** Context skill validated. Advance to post-test analysis. If not triggered: Redesign branching questions or requirement summary for clarity.

---

## Test Questions & Expected Answers

### Question 1: Did the branching questions help applicant identify their situation?

**Test:** Show applicant the context summary ("You're renewing for your child, age 7"). Ask: "Does this match what you meant to apply for?"

**Expected answer:** "Yes, that's correct" OR if no: "No, I actually meant [different scenario]" — applicant can correct if needed.

**Scoring:** ✓ If applicant correctly identified their situation. ✗ If applicant says "I'm not sure" or "I don't know what any of this means" (question clarity failed).

---

### Question 2: Does applicant understand what documents their situation needs?

**Test:** Show applicant the requirement summary for their context. Ask: "Looking at this list, does it match what you expected to bring?"

**Expected answer:** Applicant identifies at least one gap OR confirms they expected all of it. Example: "I didn't know I needed the birth certificate. I thought the photo and my ID were enough." OR "Yes, I expected all of this."

**Scoring:** ✓ If applicant can articulate the context-specific requirements (what's different for child renewal vs adult renewal). ✗ If applicant says "I don't know what any of this means."

---

### Question 3: When shown missing or invalid documents, does applicant understand why?

**Test:** Upload documents. Validation flags "Child birth certificate: Not submitted" and "Photo: Outside 3-month window." Ask: "Why do you think the system flagged these?"

**Expected answer:** Applicant restates the flag in their own words. Example: "I didn't upload the birth certificate" OR "My photo is too old."

**Scoring:** ✓ If applicant restates the issue. ✗ If applicant says "The system is broken" or "I don't know why it's rejecting this" (feedback clarity failed).

---

### Question 4: When presented with "Go Back & Fix" vs "Submit Anyway," which does applicant choose?

**Test:** Observe applicant's button selection after seeing flagged documents.

**Expected answer:** Applicant clicks "Go Back & Fix." If applicant clicks "Submit Anyway," ask why.

**Scoring:** ✓ If ≥75% of applicants (across all users) choose "Go Back & Fix." ✗ If ≥50% choose "Submit Anyway" — assumption is broken, requirements feedback doesn't change behavior.

**Recording:** Track outcome by context (parent-with-child, adult-renewal). If parent-with-child shows ≥75% fix rate but adult-renewal shows <50%, requirement clarity varies.

---

### Question 5: Can applicant describe what they'll prepare without looking back?

**Test:** After reading the summary, hide it and ask: "Walk me through what documents you'll gather before you submit. Don't look back at the list."

**Expected answer:** Applicant lists ≥2 of 3 core requirements for their context. Example (child renewal): "I need the child's birth certificate and my own ID, and then I need to get the child's photo taken."

**Scoring:** ✓ Pass if applicant accurately describes ≥2 of 3 core items. ✗ Fail if applicant lists generic items ("a photo and some paperwork") without specificity.

**This is the Week 4 clarity gate:** ≥75% of users must pass this question.

---

## Accuracy & Versioning

**Context skill accurate as of:** 20 November 2026  
**Applicant scenarios tested:** Renewal-child, renewal-self  
**Edge cases flagged for escalation:** Overseas applicant, deceased parent, name change during passport validity, applicant unsure of their own situation  
**Next review:** After Week 4 usability test (target: 25 November 2026)
