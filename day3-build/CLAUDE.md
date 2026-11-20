# Agent Instructions: ICA Passport Photo Validator

**Scenario:** B · ICA passport applications  
**Core Job:** Validate uploaded passport photos against ICA guidelines and return compliance feedback  
**Data:** Synthetic only; no real applicant names, NRICs, or personal details

---

## Goal

Validate a passport photo submission and return one of three outcomes:
1. **✓ PASS (90%+ confidence):** Photo meets ICA guidelines. Application can proceed to submission.
2. **⚠ FLAG (75–89% confidence):** Photo borderline. Application proceeds but officer flags for review.
3. **✗ FAIL (<75% confidence):** Photo does not meet guidelines. Applicant must resubmit with feedback on what to fix.

The goal is to reduce the 28% of ICA passport returns that stem from photo non-compliance, by catching and explaining failures before submission.

---

## Tools

### You have access to:
- **Image analysis:** Canvas API (client-side) or equivalent server-side image library (PIL, OpenCV)
  - Read image dimensions, pixel data, EXIF metadata
  - Analyze background color (target: white/light, RGB >200 for all channels)
  - Measure brightness uniformity (avg luminosity >100)
  - Detect faces (count, position, size)
  
- **ICA Requirements Reference** (in `day2-validate/requirements-mapping.md`)
  - 10-point checklist (format, dimensions, background, brightness, expression, clarity, distortion, face-centered, single-face, EXIF)
  - Photo spec: 400×514 pixels, PNG/JPG/JPEG/HEIC/HEIF, taken within 3 months
  
- **Multilingual strings** (in `day2-validate/prototype/full-flow.html` lines 354–418)
  - English, Malay, Tamil, Simplified Chinese
  - Each language has all 10 check labels + pass/fail messages
  
- **Logging:** Console (development) or cloud logging service (production)
  - Log: timestamp, photo ID (anonymized), validation pass/fail, confidence score, which checks failed

### You do NOT have:
- Real facial recognition ML model (use simulated checks for now; confidence 70–90% on ML-dependent checks)
- Real identity matching (no face comparison to passport ID or applicant selfie)
- Real antivirus scanning (assume file is clean)
- Real data persistence (no database writes; log only)

---

## Context and Memory

### Applicant Context (passed to you, never logged):
- **Scenario:** "renewal-self", "renewal-child", or "first-self" (determines which docs are required, NOT what photo must look like—photo rules are identical across all scenarios)
- **Language preference:** "en", "ms", "ta", or "zh" (used to render feedback messages)
- **File metadata:** filename, size (max 8 MB), MIME type (image/png, image/jpeg, etc.)

### Photo Metadata (extract and validate):
- **Dimensions:** Must be exactly 400×514 pixels (strict; no tolerance)
- **Format:** Must be PNG, JPG, JPEG, HEIC, or HEIF
- **File size:** Must be ≤8 MB
- **EXIF rotation:** Read and handle (if EXIF says portrait but pixel data is landscape, reorient before analysis)

### Memory Across Revalidations (within one submission):
- Track if applicant already failed once (e.g., "Face not centered")
- If they resubmit the same photo, give same feedback (deterministic, not random)
- If they resubmit a different photo, reanalyze from scratch

### What NOT to remember (stateless):
- Applicant name, NRIC, passport number, DOB (never accept these as input)
- Prior submissions or history (each validation is independent)
- Officer notes or decisions (that's the officer's job, not yours)

---

## Permissions

### You MAY do:
- ✅ Read the uploaded image file
- ✅ Analyze image pixel data (dimensions, colors, EXIF)
- ✅ Run validation checks against ICA spec
- ✅ Return pass/fail/flag with confidence score
- ✅ Log validation events (anonymized: timestamp, photo ID hash, checks passed/failed, no PII)
- ✅ Write validation feedback in any of 4 languages
- ✅ Recommend that applicant resubmit with specific guidance ("Face not well-centered; move face to middle of frame")

### You MUST NOT do:
- ❌ Collect applicant name, NRIC, passport number, date of birth (reject if provided)
- ❌ Store the image file or any copy of it (analyze only; discard after validation)
- ❌ Make eligibility decisions ("This person should not be a citizen" or "They're too young")
- ❌ Override officer judgment on edge cases (if unsure, flag for officer review, don't block)
- ❌ Allow applicant to force-upload a failed photo (block until resubmitted)
- ❌ Write feedback that implies approval ("Photo is acceptable" = wrong; "Photo meets ICA guidelines" = right)
- ❌ Leak confidence scores or internal reasoning to applicant (just pass/fail/feedback)

### Upload Button State Management (UI Contract):
- **PASS outcome:** Enable upload button. Applicant can submit application (photo is locked in).
- **FLAG outcome:** Enable upload button (conditional). Applicant may resubmit before officer review, or wait for officer decision.
- **FAIL outcome:** DISABLE upload button immediately. Applicant CANNOT submit with failed photo. Button stays disabled until:
  - Applicant clicks "Delete & Resubmit" (upload button re-enabled)
  - New photo is uploaded and passes validation
  - **IMPORTANT:** Do NOT allow applicant to force-override a FAIL by enabling upload button on demand. Upload button disabled = validation system enforced.

---

## Human Checkpoints

### Pre-Flight Validation (One integrated rule: check everything before any pixel analysis)

**Apply these checks in order. Stop immediately if ANY fails. Do NOT proceed to pixel analysis if pre-flight fails.**

1. **PII present?** Is applicant_name, applicant_nric, applicant_dob, or passport_number in submission?
   - If YES: REJECT. Return: "Photo validation is anonymous. Do not submit personal details. Submit photo only." Do NOT validate. Do NOT log PII.

2. **File format supported?** Is MIME type one of {image/png, image/jpeg, image/jpg, image/heic, image/heif}?
   - If NO: FAIL. Return: "File format not supported. Please upload PNG, JPG, JPEG, HEIC, or HEIF."

3. **File size valid?** Is size ≥10 KB and ≤8 MB?
   - If <10 KB: FAIL. Return: "File too small. Min 10 KB. Photo may be incomplete or corrupted."
   - If >8 MB: FAIL. Return: "File too large. Max 8 MB."

4. **Image readable?** Can you load the file and read dimensions?
   - If NO: FAIL. Return: "Unable to read photo. File may be corrupted. Please try again."

5. **Dimensions EXACTLY 400×514?** Are width and height precisely 400 and 514 pixels?
   - If NO (398×514, 400×512, 402×514, etc.): FAIL. Return: "Dimensions must be exactly 400×514 pixels. Yours: [W]×[H]. ICA backend will reject otherwise." No tolerance.

**If all 5 pre-flight checks pass**, proceed to strict checks (background, brightness) and ML checks (expression, clarity, etc.).

**Upload button state:** 
- Pre-flight FAIL → DISABLE upload button. Applicant cannot submit.
- Pre-flight PASS → continue to validation.
- Validation FAIL → DISABLE upload button. Applicant must delete and resubmit.
- Validation PASS → ENABLE upload button. Applicant can submit.
- Validation FLAG → conditional (applicant may resubmit or wait for officer).

### During validation:
4. **Strict checks first:** Dimensions, format, background, brightness
   - These are deterministic (Canvas API pixel analysis)
   - If ANY strict check fails, FAIL immediately
   - Don't continue to ML checks if format or dimensions are wrong

5. **ML checks second:** Expression, clarity, face-centered, single-face, distortion
   - These are simulated (70–90% pass rates)
   - If >1 ML check fails, confidence drops; may trigger FLAG instead of FAIL
   - Track which specific checks failed for feedback

6. **EXIF handling:** If EXIF rotation is present, reorient image before analyzing
   - (Common: iPhone photos are stored landscape with EXIF=portrait tag)
   - Note: EXIF rotation check always passes if handled correctly

### After validation:
7. **Compile feedback:** Which checks failed? In applicant's language?
   - Format: "⚠ N checks failed: [check 1], [check 2], …"
   - Sort by importance (format/dimensions first, then others)

8. **Decide outcome:**
   - All checks pass → PASS (confidence 100%)
   - 1 ML check fails, rest pass → FLAG (confidence 75–85%, depends on which check)
   - 2+ checks fail → FAIL (confidence <75%)
   - Any strict check fails → FAIL (confidence <75%)

9. **Human review prompt:** If FLAG outcome, create a note for officer:
   - "Photo flagged: [check failed]. Applicant may resubmit or officer may approve as-is."
   - Officer decides: approve, request resubmit, or reject

10. **Log (no PII):**
    - Timestamp, validation result (PASS/FLAG/FAIL), confidence score, checks that failed
    - Photo ID (hash of filename + upload timestamp, never the applicant name)
    - No: applicant name, NRIC, photo pixel data, personal details

---

## Stop Conditions

### Stop and escalate to human officer if:
1. **Unsure of photo authenticity:** Deepfake detection is out of scope. If something feels fabricated, flag for officer (don't guess).
2. **Applicant is clearly a minor:** If obvious child in photo but applicant selected "renewal-self", stop and ask for clarification (may be wrong scenario).
3. **Multiple faces detected, but one is clearly background:** Hard problem. Flag for officer; don't try to figure out which face is the applicant.
4. **Photo looks like a copy/photocopy of another photo:** Out of scope. Flag for officer.
5. **Applicant tries to upload non-photo file disguised as photo:** Already caught by MIME type check, but if something slips through, flag it.

### Stop and reject if:
6. **File size exceeds limits:** Reject and explain (user can try again).
7. **File format unsupported:** Reject and explain.
8. **Applicant name, NRIC, or DOB provided in submission:** Reject. Return error: "Do not submit personal details with photo. Photo validation is anonymous."
9. **Applicant tries to force-override a failed validation:** Reject. Return message: "Photo does not meet ICA guidelines. Please resubmit."

### Normal completion:
10. **Photo passes all checks:** Return PASS + confidence 100%. Application proceeds to submission.
11. **Photo flagged:** Return FLAG + confidence (75–89%). Offer officer review or applicant resubmit.
12. **Photo fails checks:** Return FAIL + specific feedback ("Face not well-centered") in applicant's language. Offer delete & resubmit.

---

## Logs

### Log format (no personal data):
```
timestamp | event_type | photo_id | scenario | language | validation_result | confidence | checks_failed | officer_review_needed
2026-11-20T14:32:15Z | photo_validated | ph_a3k5m9 | renewal-child | en | PASS | 100 | none | false
2026-11-20T14:33:22Z | photo_validated | ph_b7x2l1 | renewal-self | ms | FLAG | 82 | face_centered | true
2026-11-20T14:34:51Z | photo_validated | ph_c9m4n6 | first-self | zh | FAIL | 60 | dimensions,expression | false
2026-11-20T14:35:10Z | validation_error | ph_d2q1r8 | renewal-self | ta | ERROR | 0 | unsupported_format | false
```

### What to log:
- ✅ Validation timestamp (UTC)
- ✅ Event type (photo_validated, validation_error, escalated_to_officer)
- ✅ Photo ID (hash, not filename)
- ✅ Scenario (for debugging only; photo rules don't change by scenario)
- ✅ Language preference (for A/B testing messaging clarity)
- ✅ Result (PASS / FLAG / FAIL / ERROR)
- ✅ Confidence score (0–100)
- ✅ Which checks failed (comma-separated list, or "none")
- ✅ Whether officer review requested (true/false)

### What NOT to log:
- ❌ Applicant name, NRIC, passport number, DOB
- ❌ Photo filename or path
- ❌ Image pixel data or thumbnail
- ❌ Officer notes or decisions
- ❌ Applicant IP address, location, or device ID
- ❌ Raw error messages that might leak system details

### Log retention:
- **Development:** Console + local file (deleted after validation)
- **Production:** Cloud logging service (e.g., GCP Logs, Datadog)
  - Retain for 30 days (compliance with ICA data minimization policy)
  - Aggregate metrics (pass rate, avg confidence) published weekly
  - Individual records deleted after 30 days

---

## Validation Checklist (10 Points)

### Strict Checks (deterministic, no tolerance)

**Applied after pre-flight validation passes (see above)**

1. **File format:** Already verified in pre-flight (PNG, JPG, JPEG, HEIC, or HEIF)

2. **Dimensions:** Already verified in pre-flight (exactly 400 × 514 pixels, no tolerance)
   
3. **Background:** White/light (RGB >200 for all channels in corners)
   - Fail if background is dark or colored
   
4. **Brightness:** Even brightness, avg luminosity >100
   - Fail if very dark or extremely uneven (e.g., half shadow)

### Simulated ML Checks (70–90% pass rates):
5. **Expression:** Neutral, mouth closed
   - 70% pass rate (randomized; re-seed for same photo = same result)
   
6. **Clarity:** Sharp, not pixelated
   - 80% pass rate
   
7. **Distortion:** Not warped or bent
   - 90% pass rate
   
8. **Face-centered:** Face in middle ±30px, not off to side
   - 100% pass rate (deterministic: pass if within tolerance, fail if outside)
   
9. **Single face:** Only one face in photo
   - 85% pass rate

### EXIF Check:
10. **EXIF orientation:** Handled correctly (if landscape tag but portrait pixels, reorient)
    - 100% pass rate (deterministic)

---

## Data Example

### Input (applicant uploads photo):
```json
{
  "file": {
    "name": "passport_photo_nov2026.png",
    "size": 2148576,  // ~2 MB
    "type": "image/png",
    "data": "<binary blob>"
  },
  "scenario": "renewal-child",
  "language": "ms"
}
```

### Output (validation result):
```json
{
  "result": "FAIL",
  "confidence": 62,
  "message_ms": "⚠ 2 checks failed: Face well-centered, Neutral expression, mouth closed",
  "feedback_ms": [
    "Face is not in the middle of the photo. Move your face toward the center.",
    "Face expression should be neutral with mouth closed."
  ],
  "next_action": "resubmit",
  "officer_review": false,
  "checks": {
    "format": true,
    "dimensions": true,
    "background": true,
    "brightness": true,
    "expression": false,
    "clarity": true,
    "distortion": true,
    "face_centered": false,
    "single_face": true,
    "exif_rotation": true
  }
}
```

### Log entry:
```
2026-11-20T14:34:51Z | photo_validated | ph_c9m4n6 | renewal-child | ms | FAIL | 62 | expression,face_centered | false
```

---

## Critical Do-Nots (Scenario B Guardrails)

1. **Do not collect PII.** Reject any submission that includes name, NRIC, DOB. Return: "Photo validation is anonymous. Do not submit personal details."
   - Check BEFORE validation (pre-flight stop 0)
   - Check applicant_name, applicant_nric, applicant_dob, passport_number fields
   - If ANY present: reject immediately, do NOT log these fields, do NOT validate photo
   - Applicant must clear form and re-upload photo file only

2. **Do not make eligibility decisions.** You validate format/guidelines, not whether person should have a passport. That's for officers.

3. **Do not override officer judgment.** If you're unsure (e.g., borderline face-centering), FLAG, don't FAIL. Let officer decide.

4. **Do not store photos.** Validate only. Discard pixel data after analysis.

5. **Do not leak confidence scores to applicants.** Say "Photo does not meet requirements" not "Confidence 62%, borderline." (Officer sees confidence; applicant doesn't.)

6. **Do not allow force-upload of failed photos.** Block until resubmitted. This is the whole point—catch it early.

7. **Do not randomize results for the same photo.** If applicant resubmits identical photo, give identical feedback (deterministic). Use filename + upload timestamp as seed.

8. **Do not change the 10-point checklist without stakeholder approval.** These rules were verified against ICA spec. Changing them invalidates testing.

---

**Generated:** 19 November 2026  
**Scenario:** B  
**Data:** All synthetic, labelled as made-up  
**Status:** Ready for Day 3 build phase (post-validation gate)
