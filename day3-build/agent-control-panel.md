# Agent Control Panel: ICA Passport Photo Validator

**Agent:** Photo Validator (CLAUDE.md)  
**Scenario:** B · ICA passport applications  
**Core mission:** Validate passport photos against ICA compliance rules in real time; catch non-compliant photos before submission to reduce returns.  
**Data:** Synthetic only; no real applicant names, NRICs, or personal details.

---

## What the Agent Is Doing

**In one sentence:** The agent receives an uploaded photo, validates it against a 10-point ICA checklist in ~1–2 seconds, and returns PASS (proceed to submit), FLAG (officer reviews), or FAIL (applicant resubmits) with specific feedback.

**The job:**
1. Verify file is readable (not corrupted, wrong format)
2. Run 10-point checklist: 4 strict geometric checks + 5 ML-simulated checks + 1 EXIF check
3. Assign confidence score (0–100%)
4. Decide outcome: PASS (90%+), FLAG (75–89%), FAIL (<75%)
5. Return feedback in applicant's language with guidance on what to fix
6. Create officer note if FLAG (for officer review workflow)
7. Log validation event anonymously (no PII, no photo stored)

**Success:** Applicant knows exactly why photo passed or failed; can resubmit if needed; officer gets structured note for borderline cases.

---

## Opportunity Portfolio

**What decisions does the agent encounter?**

### 1. Pre-flight (acceptance or rejection, agent alone)
- **File format supported?** (PNG, JPG, JPEG, HEIC, HEIF)
  - Alone: Reject unsupported format with explanation
- **File size within 8 MB?**
  - Alone: Reject if over limit with explanation
- **File readable?** (not corrupted, truncated)
  - Alone: Reject if unreadable with explanation

### 2. Strict Checks (deterministic, agent alone)
- **Exact 400×514 pixels?**
  - Alone: FAIL if not exact (no tolerance)
- **White/light background?** (RGB >200 all channels)
  - Alone: FAIL if dark or colored
- **Brightness even?** (avg luminosity >100)
  - Alone: FAIL if very dark or uneven

### 3. ML Checks (simulated, agent alone with confidence scoring)
- **Neutral expression, mouth closed?** (70% pass rate)
  - Alone: Include in confidence calc; may trigger FLAG if fails
- **Sharp, not pixelated?** (80% pass rate)
  - Alone: Include in confidence calc
- **No distortion/warping?** (90% pass rate)
  - Alone: Include in confidence calc
- **Face centered within ±30px?** (100% deterministic)
  - Alone: PASS if within tolerance, FAIL if outside
- **Single face only?** (85% pass rate)
  - Alone: Include in confidence calc

### 4. Outcome Decision (agent alone, conditional escalation)
- **All 10 checks pass?** → PASS (confidence 100%)
  - Alone: Return PASS + proceed to submission
- **1 ML check fails, rest pass?** → FLAG (confidence 75–89%)
  - Alone: Return FLAG + create officer note + allow submission with warning
- **2+ checks fail or any strict check fails?** → FAIL (confidence <75%)
  - Alone: Return FAIL + resubmit request + specific feedback
- **Unsure of authenticity / suspected deepfake?** → Escalate
  - Proposes: Flag for officer with "authenticity uncertain" note

### 5. Logging (audit, agent alone)
- **Log validation event** (timestamp, photo ID hash, result, confidence, checks failed)
  - Alone: Log with no PII, no photo data

---

## Rules of Engagement

### Agent Decides Alone (No Human Required)

1. **File pre-flight checks** (format, size, readability)
   - Reject with user-friendly error message if any fail
   - No officer involvement needed

2. **Strict geometric checks** (dimensions 400×514, background, brightness)
   - Apply deterministically (same photo = same result every time)
   - FAIL immediately if any strict check fails
   - No flexibility or override; matches ICA backend expectations

3. **ML check outcomes** (expression, clarity, distortion, face-centered, single-face)
   - Run simulated checks with fixed pass rates (70–90%)
   - Include results in confidence calc
   - Use filename + upload timestamp as seed (determinism for resubmits)

4. **Outcome determination** (PASS / FAIL with confidence)
   - If all 10 pass → PASS (100%)
   - If 2+ fail or any strict fails → FAIL (<75%)
   - Return specific feedback on which checks failed, in applicant's language

5. **Logging** (anonymized, no PII)
   - Log every validation: timestamp, photo ID (hash), scenario, language, result, confidence, checks failed
   - Never log: applicant name, NRIC, photo data, device ID, IP address

---

### Agent Proposes & Waits (Officer Decides)

1. **FLAG outcome** (1 ML check fails, rest pass; confidence 75–89%)
   - Agent: Return FLAG + create structured note:
     - "Photo flagged: [which check]. Applicant may resubmit or officer may approve."
     - Example: "Face 15px off-center, within tolerance but borderline. Applicant may resubmit or officer may approve."
   - Officer: Reviews note + photo + context, decides: approve, request resubmit, or reject
   - Applicant: Can see FLAG message, resubmit if they choose, or wait for officer

2. **Suspected authenticity issues** (deepfake, photocopy, multiple faces with unclear primary)
   - Agent: Flag with "Authenticity uncertain" + escalate to officer
   - Officer: Manually inspect, approve or reject
   - Applicant: Told "Photo under review; officer will contact you if clarification needed"

3. **Edge case scenarios** (overseas, lost passport, name change)
   - Agent: Detects scenario selection in Step 1B, escalates (no photo validation attempted for these)
   - Officer: Handles manually per ICA business process
   - Applicant: Redirected to ICA contact page

---

### Person Reviews (Oversight, Not Overridden)

1. **Weekly validation pattern audit**
   - Review aggregate metrics: pass rate, average confidence, which checks most frequently fail
   - Flag if patterns shift unexpectedly (e.g., brightness failures spike → possible lighting change in office, or model drift)
   - Decide: continue or pause validation, adjust pass rates if needed

2. **Monthly accuracy spot-check**
   - Sample 10–20 photos across result categories (PASS, FLAG, FAIL)
   - Manually verify against ICA spec: did agent decide correctly?
   - If accuracy <90%: investigate and retrain/adjust

3. **Officer feedback loop**
   - Collect officer comments on FLAG cases: "Did the note help?" "Did you approve/reject based on note?"
   - Feed into next iteration: improve officer note clarity if pattern emerges

4. **PII/privacy audit**
   - Monthly check of logs: confirm no applicant data accidentally logged
   - Annual security review with ICA privacy team

---

## Stop Conditions

### Agent Stops and Rejects (No Submission)

1. **File is corrupted or unreadable**
   - Message: "Unable to read photo. File may be corrupted. Please try again."
   - Action: User re-uploads

2. **Unsupported file format**
   - Message: "File format not supported. Please upload PNG, JPG, JPEG, HEIC, or HEIF."
   - Action: User re-uploads in supported format

3. **File exceeds 8 MB**
   - Message: "File too large. Max 8 MB. Please try a smaller file."
   - Action: User re-uploads smaller version

4. **Applicant submits PII** (name, NRIC, DOB, passport number in submission fields)
   - Message: "Photo validation is anonymous. Do not submit personal details. Submit only the photo file."
   - Action: Form cleared, user re-uploads photo only

5. **Applicant tries force-override a failed photo**
   - Message: "Photo does not meet ICA requirements. Please resubmit."
   - Action: Upload button disabled until successful resubmit

6. **FAIL outcome** (confidence <75%, 2+ checks fail)
   - Message: "❌ 2 checks failed: [check 1], [check 2]. Please resubmit."
   - Action: Upload blocked until re-upload succeeds

---

### Agent Stops and Escalates (Officer Takes Over)

1. **Suspected deepfake or photo tampering**
   - Agent: Creates escalation note with flagged pixel patterns or anomalies
   - Officer: Manual review, approve or reject
   - Escalation reason: Out of scope; requires human judgment

2. **Authenticity uncertain** (e.g., photo looks like photocopy, or printed photo photographed)
   - Agent: Flags "Authenticity uncertain" + escalates
   - Officer: Inspects, decides
   - Escalation reason: Pattern-recognition limits; may be valid edge case

3. **Multiple faces, unclear which is applicant**
   - Agent: Flags "Multiple faces detected; unclear which is applicant" + escalates
   - Officer: Clarifies with applicant, approves correct photo or rejects if ambiguous
   - Escalation reason: Ambiguous; agent cannot safely guess

4. **FLAG outcome** (1 check fails; confidence 75–89%)
   - Agent: Submits application with FLAG note for officer
   - Officer: Reviews, approves or rejects
   - Escalation reason: Borderline; need human judgment

5. **Edge case scenario** (overseas, lost passport, name change)
   - Agent: Detects in Step 1B, does not attempt photo validation
   - Officer: Handles per manual process
   - Escalation reason: Outside scope of automation; requires special handling

---

## What Stays Human

### Decisions Only Humans Make

1. **Eligibility determination**
   - ❌ Agent does NOT decide: "This person is eligible for a passport" or "They're too young"
   - ✅ Officer decides: Based on full application, supporting docs, ICA rules

2. **Special circumstances override**
   - ❌ Agent does NOT override: "Exceptional case, approve anyway"
   - ✅ Officer decides: Based on policy, discretion, applicant explanation

3. **Photo authenticity judgment**
   - ❌ Agent does NOT decide: "This is a real person" or "This is fake"
   - ✅ Officer decides: Based on human inspection, training, context

4. **Application approval/rejection**
   - ❌ Agent does NOT decide: "Application is approved"
   - ✅ Agent decides: "Photo validation: PASS / FLAG / FAIL"
   - ✅ Officer decides: "Application approved" (after full review)

5. **System rule changes**
   - ❌ Agent does NOT change: "I'll now accept 398×512 instead of 400×514"
   - ✅ Human stakeholder decides: ICA + SWE team approve, then update CLAUDE.md

6. **Photo validation rules**
   - ❌ Agent does NOT decide: "Background brightness threshold is now 95 instead of 100"
   - ✅ Human stakeholder decides: ICA + SWE team verify new rule against spec, update code

7. **User support decisions**
   - ❌ Agent does NOT escalate: "User seems frustrated, let them override"
   - ✅ Officer / support team decides: Based on user contact, context, policy

---

## Escalation Matrix (at a glance)

| Decision | Agent | Officer | Notes |
|----------|-------|---------|-------|
| File format unsupported | ✅ Reject | — | Pre-flight; no submission |
| File too large | ✅ Reject | — | Pre-flight; no submission |
| File corrupted | ✅ Reject | — | Pre-flight; no submission |
| Dimensions 400×514 exact | ✅ FAIL if not | — | Strict; no tolerance |
| Background white | ✅ FAIL if not | — | Strict; deterministic |
| All 10 checks pass | ✅ PASS | — | Submit directly |
| 1 ML check fails (75–89% confidence) | ✅ FLAG | ✅ Reviews + decides | Borderline; applicant may resubmit or wait |
| 2+ checks fail (<75% confidence) | ✅ FAIL | — | User resubmits |
| Suspected deepfake / tampering | ✅ Flag + escalate | ✅ Approves/rejects | Out of scope for agent |
| Multiple faces, unclear primary | ✅ Flag + escalate | ✅ Clarifies | Ambiguous; need human judgment |
| Applicant submits PII | ✅ Reject | — | Privacy guardrail; re-upload |
| Application eligibility | — | ✅ Decides | Not agent's role |
| Special circumstances override | — | ✅ Decides | Human judgment only |

---

## Health Checks (Operational Oversight)

**Weekly:**
- Pass rate (% PASS / total validations)
- Confidence distribution (avg, min, max)
- Top 3 failing checks
- Officer override rate on FLAG cases

**Monthly:**
- Accuracy spot-check (sample 10–20, verify against ICA spec)
- PII audit (logs contain no personal data)
- Error rate (rejections due to technical issues, not rule violations)

**Quarterly:**
- Benchmark against real ICA photo compliance rates (if available)
- Applicant trust survey (do they understand error messages?)
- Officer feedback (are FLAG notes useful?)

---

**Generated:** 19 November 2026  
**Scenario:** B  
**Status:** Ready for Day 3 build + usability testing (Week 4)
