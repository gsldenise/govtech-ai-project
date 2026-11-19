# AI PRD: ICA Passport Photo Validator

**Date:** 19 November 2026  
**Scenario:** B · ICA passport applications  
**Core Job:** Validate uploaded passport photos against ICA guidelines and return compliance feedback  
**Data:** Synthetic only; no real applicant names, NRICs, or personal details

---

## Problem and User

**User:** ~520,000 applicants per year uploading passport photos; ICA officers reviewing submissions.

**The problem:**
- 28% of ICA returns stem from photo non-compliance (wrong dimensions, background, expression, clarity)
- Applicants don't know photo requirements upfront and often submit multiple times
- ICA officers waste time rejecting photos that fail basic checks
- Applicant experience: rejection + resubmit + 6-day additional wait = frustration

**The opportunity:**
- Catch non-compliant photos *before* ICA submission (client-side, in real-time)
- Return specific feedback so applicants know exactly what to fix
- Reduce photo-related returns from 28% to <15%, saving ICA ~6 days per applicant

**Success looks like:**
- Applicant uploads photo → agent validates in ~1–2 seconds
- Applicant sees: ✓ Photo passes (proceed) OR ✗ Photo fails + specific reason (e.g., "Face not centered; move to middle of frame")
- Officer sees FLAG cases (75–89% confidence) with structured notes for final decision
- Return rate on photo drops from 28% to <15%

---

## Instructions

**Agent role:** Real-time photo compliance validator (client-side, embedded in submission workflow).

**The workflow:**
1. **Applicant uploads passport photo** → FileReader loads image
2. **Agent runs 10-point checklist:**
   - 4 strict checks (format, dimensions, background, brightness) — deterministic, fail-fast
   - 5 ML checks (expression, clarity, distortion, face-centered, single-face) — simulated (70–90% pass rates)
   - 1 EXIF check (rotation handling) — deterministic
3. **Agent assigns confidence score** (0–100%) based on which checks pass/fail
4. **Agent decides outcome:**
   - PASS (90%+) → applicant proceeds to submit application
   - FLAG (75–89%) → application submits with officer-review flag; applicant can resubmit or wait
   - FAIL (<75%) → upload blocked; applicant must resubmit with feedback on what to fix
5. **Agent returns feedback** in applicant's language (English, Malay, Tamil, Simplified Chinese)
6. **Agent logs validation event** (timestamp, photo ID hash, result, confidence, checks failed — no PII)
7. **If FLAG, agent creates officer note:** "Photo flagged: [check failed]. Applicant may resubmit or officer may approve."

**Key design principle:** Deterministic on geometric checks (applicant can trust the same photo always passes), simulated on ML checks (transparent about limitations).

---

## Tools

**Agent has access to:**
- **Canvas API (client-side)** or equivalent (PIL, OpenCV on backend)
  - Read image dimensions, pixel data, EXIF metadata
  - Analyze background color (target: white/light, RGB >200 for all channels in corners)
  - Measure brightness uniformity (average luminosity >100)
  - Detect faces (count, position, bounding box)
  
- **ICA Requirements Reference** (requirements-mapping.md)
  - 10-point checklist verified against ICA official spec
  - Photo spec: 400×514 pixels, PNG/JPG/JPEG/HEIC/HEIF, taken within 3 months
  
- **Multilingual strings** (full-flow.html lines 354–418)
  - All 10 check labels + pass/fail messages in 4 languages
  - Language parameter: "en", "ms", "ta", "zh"
  
- **Logging service** (console in dev, cloud logging in production)
  - Timestamp, photo ID (hash), validation result, confidence, checks failed
  - No PII, no photo data stored

**Agent does NOT have:**
- Real facial recognition ML model (use simulated checks; 70–90% pass rates represent unavailable ML)
- Real identity matching (no face-to-ID comparison)
- Antivirus/malware scanning (assume file is clean)
- Database write access (logs only, no persistence)
- Singpass integration (handled by separate step)

---

## Context and Memory

**Applicant context (passed to agent, never logged):**
- **Scenario:** "renewal-self", "renewal-child", or "first-self"
  - Determines which documents are required, NOT photo validation rules (photo rules identical across all scenarios)
- **Language preference:** "en", "ms", "ta", "zh" (used to render feedback in applicant's language)
- **File metadata:** filename, size (max 8 MB), MIME type (image/png, image/jpeg, image/jpg, image/heic, image/heif)

**Photo metadata (extract, validate, use):**
- **Dimensions:** Must be exactly 400×514 pixels (strict; no ±5% tolerance — backend will reject anyway)
- **Format:** Must be one of {PNG, JPG, JPEG, HEIC, HEIF}
- **File size:** Must be ≤8 MB
- **EXIF rotation:** If present, reorient pixel data before analysis (e.g., iPhone stores landscape with portrait EXIF tag)

**Memory within one submission (determinism for resubmits):**
- If applicant resubmits the same photo file (same filename + timestamp), return same validation result
- Use filename + upload timestamp as seed for simulated ML checks (ensures repeatability)
- If applicant uploads different photo, reanalyze from scratch

**What agent does NOT remember (stateless):**
- Applicant name, NRIC, passport number, DOB (reject if provided)
- Prior submissions or validation history
- Officer notes or decisions on edge cases
- Any personally identifiable information

---

## Permissions

**All actions listed below with proposed tier and setting. For each, review and update to auto/confirm/block as appropriate.**

### Read Actions

| Action | Description | Tier | Setting | Proposed |
|--------|-------------|------|---------|----------|
| Read uploaded photo file | Load image via FileReader; read pixel data | Read | Auto | ✓ |
| Extract image dimensions | Canvas API: image.width, image.height | Read | Auto | ✓ |
| Extract EXIF metadata | Read EXIF rotation tag from photo header | Read | Auto | ✓ |
| Analyze pixel colors | Sample RGB values from corners + face region | Read | Auto | ✓ |
| Measure brightness | Calculate average luminosity across image | Read | Auto | ✓ |
| Detect faces | Identify face regions (count, position, bounding box) | Read | Auto | ✓ |
| Access ICA spec reference | Read requirements-mapping.md (static) | Read | Auto | ✓ |
| Access multilingual strings | Read language labels from full-flow.html (static) | Read | Auto | ✓ |

### Draft Actions

| Action | Description | Tier | Setting | Proposed |
|--------|-------------|------|---------|----------|
| Generate validation feedback | Compile plain-language reason for pass/fail/flag | Draft | Auto | ✓ |
| Create officer note | For FLAG cases, write structured note for human review | Draft | Confirm | ✓ |
| Calculate confidence score | Compute 0–100% confidence based on checks passed | Draft | Auto | ✓ |
| Decide outcome | PASS (90%+) / FLAG (75–89%) / FAIL (<75%) | Draft | Auto | ✓ |

### Write Actions

| Action | Description | Tier | Setting | Proposed |
|--------|-------------|------|---------|----------|
| Display validation result to applicant | Show ✓/✗ for each of 10 checks; overall result | Write | Auto | ✓ |
| Update upload button state | Disable upload if photo fails; enable if passes | Write | Auto | ✓ |
| Show error message (user-facing) | "Photo does not meet ICA requirements" in applicant language | Write | Auto | ✓ |
| Render photo preview | Display uploaded photo alongside validation results | Write | Auto | ✓ |
| Log validation event | Write: timestamp, photo ID, scenario, language, result, confidence, checks failed | Write | Auto | ✓ |

### Send Actions

| Action | Description | Tier | Setting | Proposed |
|--------|-------------|------|---------|----------|
| Submit validation log to server | POST anonymous validation event to cloud logging | Send | Confirm | ✓ |
| Escalate FLAG to officer queue | Route FLAG case + officer note to officer review workflow | Send | Confirm | ✓ |
| Return validation result to application form | JSON: {result, confidence, message, feedback, checks, officer_note} | Send | Auto | ✓ |

### Blocked Actions

| Action | Description | Rationale |
|--------|-------------|-----------|
| Collect applicant name/NRIC/DOB | Anonymous validation only | ❌ Block |
| Store photo file (copy/archive) | Analyze only; discard after validation | ❌ Block |
| Make eligibility decision | "Person should/should not have passport" — not agent's role | ❌ Block |
| Override officer judgment | FLAG means human decides, not agent override | ❌ Block |
| Allow force-upload of failed photo | Upload must be blocked until resubmitted | ❌ Block |
| Change validation rules without stakeholder approval | Rules locked to ICA spec; only humans can change | ❌ Block |
| Leak confidence scores to applicants | Applicants see PASS/FAIL only, not internal scores | ❌ Block |
| Make randomized decisions on geometric checks | Face-centered, dimensions, background must be deterministic | ❌ Block |

---

## Human Checkpoints

**Checkpoints baked into agent logic (non-negotiable):**

1. **Pre-flight verification** (before pixel analysis)
   - ✓ Verify MIME type ∈ {image/png, image/jpeg, image/jpg, image/heic, image/heif}
     - If not → REJECT: "File format not supported. Please upload PNG, JPG, JPEG, HEIC, or HEIF."
   - ✓ Verify file size ≤8 MB
     - If not → REJECT: "File too large. Max 8 MB."
   - ✓ Verify image readable (not corrupted)
     - If not → REJECT: "Unable to read photo. File may be corrupted. Please try again."

2. **Strict checks first** (deterministic, no tolerance)
   - ✓ Dimensions exactly 400×514? → FAIL if not
   - ✓ Background white/light (RGB >200 all channels)? → FAIL if dark
   - ✓ Brightness even (avg >100)? → FAIL if very dark
   - If ANY strict check fails → STOP, return FAIL immediately (don't continue to ML checks)

3. **ML checks second** (simulated, included in confidence calc)
   - ✓ Expression neutral, mouth closed? (70% pass rate)
   - ✓ Clarity sharp, not pixelated? (80% pass rate)
   - ✓ Distortion none? (90% pass rate)
   - ✓ Face centered within ±30px? (100% deterministic: PASS if within, FAIL if outside)
   - ✓ Single face only? (85% pass rate)
   - Track which fail for feedback

4. **Confidence calculation & outcome decision**
   - All 10 pass → PASS (confidence 100%)
   - 1 ML fails + 9 pass → FLAG (confidence 75–89%, depending on which check)
   - 2+ fail OR any strict fails → FAIL (confidence <75%)

5. **Feedback generation** (specific, actionable, in applicant's language)
   - List which checks failed (e.g., "Face not centered, expression not neutral")
   - Suggest fix (e.g., "Move face to center of photo; try a neutral expression with mouth closed")
   - Sort by importance (format/dimensions first, then others)

6. **EXIF handling**
   - If EXIF rotation present, reorient pixel data before analysis
   - Note: Always passes if handled correctly

7. **Officer note generation (FLAG only)**
   - "Photo flagged: [check failed]. Position: [specific issue]. Applicant may resubmit or officer may approve."
   - Example: "Face 15px off-center, within tolerance but borderline. Applicant may resubmit or you may approve."
   - Officer decides: approve, request resubmit, or reject

---

## Stop Conditions

### Stop and Reject (No Submission)

1. **Unsupported file format**
   - Message: "File format not supported. Please upload PNG, JPG, JPEG, HEIC, or HEIF."
   - User action: Re-upload in correct format

2. **File too large**
   - Message: "File too large. Max 8 MB."
   - User action: Compress or re-upload smaller file

3. **File corrupted / unreadable**
   - Message: "Unable to read photo. File may be corrupted. Please try again."
   - User action: Re-upload or take new photo

4. **PII submitted** (name, NRIC, DOB, passport number in submission fields)
   - Message: "Photo validation is anonymous. Do not submit personal details. Submit only the photo file."
   - User action: Clear form and re-upload photo only

5. **FAIL outcome** (confidence <75%, 2+ checks fail or any strict check fails)
   - Message: "❌ [N] checks failed: [check 1], [check 2]. [Specific fix guidance]. Please resubmit."
   - User action: Delete current photo and re-upload corrected photo

6. **Applicant tries force-override**
   - Message: "Photo does not meet ICA requirements. Please resubmit."
   - User action: Upload button remains disabled until successful resubmit

---

### Stop and Escalate (Officer Takes Over)

1. **FLAG outcome** (confidence 75–89%, 1 ML check fails)
   - Message to applicant: "⚠ Photo borderline. You may resubmit or wait for officer review."
   - Application submits with FLAG + officer note
   - Officer reviews: approves, rejects, or requests resubmit

2. **Suspected authenticity issues** (deepfake, photocopy, multiple faces with unclear primary)
   - Agent flags: "Authenticity uncertain" + escalates to officer
   - Officer inspects and decides
   - Applicant told: "Photo under review; officer will contact you if clarification needed"

3. **Multiple faces detected, unclear which is primary**
   - Agent flags + escalates
   - Officer clarifies with applicant
   - Applicant re-uploads or confirms which face is primary

---

## Logs

**Log format (anonymized, no PII):**
```
timestamp | event_type | photo_id | scenario | language | validation_result | confidence | checks_failed | officer_review_needed
2026-11-20T14:32:15Z | photo_validated | ph_a3k5m9 | renewal-child | en | PASS | 100 | none | false
2026-11-20T14:33:22Z | photo_validated | ph_b7x2l1 | renewal-self | ms | FLAG | 82 | face_centered | true
2026-11-20T14:34:51Z | photo_validated | ph_c9m4n6 | first-self | zh | FAIL | 60 | dimensions,expression | false
2026-11-20T14:35:10Z | validation_error | ph_d2q1r8 | renewal-self | ta | ERROR | 0 | unsupported_format | false
```

**What to log:**
- ✅ Timestamp (UTC)
- ✅ Event type (photo_validated, validation_error, escalated_to_officer)
- ✅ Photo ID (hash of filename + upload timestamp, never the applicant name)
- ✅ Scenario (renewal-self, renewal-child, first-self)
- ✅ Language preference (en, ms, ta, zh)
- ✅ Result (PASS / FLAG / FAIL / ERROR)
- ✅ Confidence score (0–100)
- ✅ Checks failed (comma-separated list, or "none")
- ✅ Whether officer review needed (true/false)

**What NOT to log:**
- ❌ Applicant name, NRIC, passport number, DOB
- ❌ Photo filename or file path
- ❌ Image pixel data or thumbnail
- ❌ Officer notes or decisions
- ❌ Applicant IP, device ID, location
- ❌ Raw error messages that leak system details

**Log retention:**
- **Development:** Console + local file (deleted after validation)
- **Production:** Cloud logging service (e.g., GCP Logs, Datadog)
  - Retain for 30 days (ICA data minimization policy)
  - Aggregate metrics (pass rate, avg confidence) published weekly
  - Individual records deleted after 30 days

---

## Business Rules

**Immutable rules (no changes without ICA + SWE stakeholder approval):**

1. **Photo dimensions:** Exactly 400×514 pixels (not "within 5%", not "closest multiple")
   - Rationale: ICA backend rejects anything else; backend expectation is strict

2. **Background color:** RGB >200 for all channels in corners (white/very light)
   - Rationale: ICA spec requires white background; protects against dark/colored backgrounds

3. **Brightness:** Average luminosity >100 (even brightness, not very dark)
   - Rationale: ICA guideline; catches half-shadow, uneven lighting

4. **Allowed file formats:** PNG, JPG, JPEG, HEIC, HEIF only
   - Rationale: Canvas API + ICA backend support; no BMP, TIFF, GIF, etc.

5. **File size limit:** 8 MB
   - Rationale: Prevent abuse; reasonable for passport photo

6. **Face-centered check:** Within ±30px of image center (x ∈ [170, 230], y ∈ [227, 287] for 400×514)
   - Rationale: Applicant visible in photo, not off to side

7. **Single face requirement:** Only 1 face detected
   - Rationale: Only applicant in photo; no family members, children, or background faces

8. **ML pass rates (simulated):**
   - Expression (neutral, mouth closed): 70%
   - Clarity (sharp, not pixelated): 80%
   - Distortion (no warping): 90%
   - Face-centered (±30px): 100% deterministic
   - Single face: 85%
   - Rationale: Transparent about model limitations; 70–90% represents typical real ML accuracy

9. **Confidence bands:**
   - PASS: 90%+ confidence → proceed directly
   - FLAG: 75–89% confidence → submit with officer review
   - FAIL: <75% confidence → block upload, request resubmit
   - Rationale: Clear decision thresholds; FLAG captures borderline cases for human judgment

10. **Upload blocking:** If validation fails, upload is disabled until resubmit succeeds
    - Rationale: Catch errors early; no force-override to undermine validation

---

## What Engineering Decides

**Technical decisions (SWE team makes these):**

1. **Client-side vs. server-side validation**
   - Current: Client-side (Canvas API, JavaScript)
   - Trade-off: Fast, responsive UX (1–2 sec) vs. no guarantee photo reaches backend
   - Recommendation: Keep client-side for UX; backend re-validates as defense-in-depth

2. **Real ML model vs. simulated checks**
   - Current: Simulated (70–90% pass rates via Math.random())
   - When: Replace with real ML when trained model becomes available (Week 6+)
   - Seed: Use filename + upload timestamp as seed for resubmit determinism

3. **Where to run validation**
   - Current: Browser (embedded in form step)
   - Future: Could move to backend once real ML is available
   - Constraint: Must complete in <3 seconds for acceptable UX

4. **Logging destination**
   - Current: Console (development)
   - Future: Cloud logging service (production)
   - Requirement: No PII in logs; anonymized photo ID only

5. **Language handling**
   - Current: 4 languages hardcoded in HTML (English, Malay, Tamil, Simplified Chinese)
   - Future: Internationalization framework (i18next, react-intl) if more languages added
   - Constraint: Language parameter passed from Step 1; error messages match applicant's choice

6. **EXIF handling**
   - Implementation: If EXIF rotation tag present, reorient pixel data before analysis
   - Edge case: iPhone photos often stored landscape with portrait EXIF tag
   - Fallback: If EXIF handling fails, log error but continue with pixel data as-is

7. **Determinism for resubmits**
   - Seed: filename + upload timestamp → use as seed for simulated ML checks
   - Guarantee: Same photo file returns same validation result
   - Benefit: Applicant trust (not randomly rejected on second try)

---

## Implementation Gaps (Cannot Build From This PRD Without Clarification)

**Critical blockers — engineering lead cannot start without these specs:**

### 1. ML Check Implementation Algorithms
**Blocker:** PRD says "expression: neutral, mouth closed (70% pass rate)" but no detection method.
- How to detect expression? Pixel-level heuristics? Template matching? Pre-trained model?
- How to detect mouth open/closed? Edge detection? Color thresholding?
- Same for: clarity (sharpness), distortion (warping), single-face (face count)
- **Action needed:** Either provide detection algorithms or move to Open Decisions

**Current state:** Living PRD says simulated checks use Math.random(); that's fine for prototype but PRD doesn't document which algorithm replaces it.

---

### 2. Pixel Measurement Specifications
**Blocker:** "RGB >200 all channels in corners" — underspecified.
- Which corners? All 4? Just 1? How many pixels per corner?
- Algorithm: Median? Average? All pixels must be >200 or just threshold?
- Background color sampling: Do we allow 1% tolerance for compression artifacts?
- **Action needed:** Exact pixel sampling algorithm (e.g., "sample 100×100px from each corner, median RGB, all ≥200")

**Current state:** Prototype uses Canvas API; code not reviewed for exact thresholds.

---

### 3. Brightness Calculation Method
**Blocker:** "Average luminosity >100" — which color space?
- RGB luminosity: 0.299R + 0.587G + 0.114B? Or equal weight (R+G+B)/3?
- HSL lightness? Lab lightness?
- Entire image or just face region?
- **Action needed:** Exact formula (e.g., "RGB luminosity = 0.299*R + 0.587*G + 0.114*B; average across all pixels; threshold 100")

---

### 4. Face Detection Library
**Blocker:** Canvas API doesn't have built-in face detection.
- Which library? face.js? TensorFlow.js? OpenCV.js? AWS Rekognition?
- Client-side (browser) or server-side?
- Performance requirement: Must complete in <1 second as part of ~2 second SLA
- **Action needed:** Pick face detection library + document performance baseline

---

### 5. Exact Confidence Calculation Formula
**Blocker:** "1 ML fails = 75–89%, 2+ fails = <75%" — not precise enough for deterministic revalidation.
- If expression fails, confidence = 75%? Or depends on which other checks pass?
- If expression + clarity fail, confidence = 40%? 50%?
- Weighting: Do all checks contribute equally or do strict checks weight higher?
- **Action needed:** Exact formula (e.g., "Confidence = 100 * (checks_passed / 10); if 1 ML fails only, confidence = min(100, max(75, base_confidence))")

**Current state:** agent-run-log.md shows Run 2 at 78% confidence for 1 check failure, but no formula documented.

---

### 6. EXIF Rotation Handling Specifics
**Blocker:** "If EXIF says portrait but pixel data is landscape, reorient" — how?
- Which EXIF tags to check? (Orientation tag = 1–8)
- How to reorient? CSS transform? Canvas rotate + redraw? Image re-encoding?
- Performance impact: Does rotation add 500ms to validation time?
- Fallback: If EXIF tag corrupted, continue with raw pixel data or fail?
- **Action needed:** Exact implementation (e.g., "Read EXIF Orientation tag; if 6, rotate 90° CW via canvas.rotate() before pixel analysis")

---

### 7. Face-Centered Tolerance Window
**Blocker:** "Within ±30px of image center" — measured from where?
- Image center (200, 257) or face bounding box center?
- Tolerance window: ±30px means x ∈ [170, 230], y ∈ [227, 287]? Or circular distance?
- If face is 100×150px bounding box, is center of bounding box or geometric center?
- **Action needed:** Exact algorithm (e.g., "Face bounding box center must be within x ∈ [170, 230], y ∈ [227, 287]; use Euclidean distance or axis-aligned box?")

**Current state:** Business rule #6 says "x ∈ [170, 230], y ∈ [227, 287]" — good. Implementation detail: bounding box center or geometric center?

---

### 8. Officer Note JSON Schema
**Blocker:** "Create officer note" — what's the exact schema?
- Fields: { checked_by, timestamp, photo_id, scenario, language, failed_check, confidence, recommendation }?
- Where is note stored? Database table? Log entry? Separate file?
- How does officer access it? In a UI? Via API?
- What's the format for officer feedback? ("Please resubmit" vs. "Approved" vs. "Rejected")
- **Action needed:** JSON schema + storage location + officer UX sketch

---

### 9. Photo ID Hash Algorithm
**Blocker:** "Photo ID (hash of filename + upload timestamp)" — which hash?
- SHA-256? MD5? Something else?
- Format: hash("passport_photo.jpg" + "2026-11-20T14:32:15Z") or structured { filename, timestamp }?
- Collision risk: If two users upload "passport_photo.jpg" at same timestamp, same hash?
- **Action needed:** Exact algorithm (e.g., "photo_id = SHA256(filename + '|' + upload_timestamp_iso8601)")

---

### 10. Resubmit Seed Determinism
**Blocker:** "Use filename + upload timestamp as seed for Math.random()" — underspecified.
- How to seed JavaScript Math.random()? (It doesn't natively accept seeds)
- Do we use a seeded PRNG library (seedrandom.js)?
- Cross-session determinism: If user closes browser and uploads same file 1 hour later, different timestamp = different results. Is that acceptable?
- **Action needed:** Implementation (e.g., "Import seedrandom.js; seed with hash(filename + upload_timestamp); call Math.random() for ML checks")

---

### 11. Distortion Detection Algorithm
**Blocker:** "Distortion: no warping (90% pass rate)" — no algorithm specified.
- How to detect warping with Canvas API? (No built-in distortion detection)
- Heuristic: Line straightness? Grid pattern matching? Template correlation?
- With simulated checks, this is moot (70–90% = random). But when real ML comes, what detects distortion?
- **Action needed:** For prototype: either keep simulated or define heuristic; for production: specify ML or library

---

### 12. Logging Database Schema
**Blocker:** Shows log format as pipe-delimited string, but doesn't specify backend schema.
- Table schema: CREATE TABLE validation_logs (id, timestamp, photo_id, scenario, language, result, confidence, checks_failed, officer_review_needed)?
- Indexing: Which fields are indexed for officer queries?
- Data types: confidence as INT or FLOAT? checks_failed as TEXT or JSON?
- Retention: How to delete 30+ day logs automatically? TTL? Scheduled job?
- **Action needed:** Database schema + index definitions + retention query

---

### 13. Form Integration JSON Schema
**Blocker:** "Return validation result to application form" — exact JSON not specified.
- Does form expect: { result, confidence, message, feedback, checks, officer_note } or different structure?
- How does form respond to result? Enable/disable submit button? Show/hide UI?
- Error cases: What if validation times out? Return { result: "ERROR", error: "timeout" }?
- **Action needed:** Exact JSON schema with all possible response types

---

### 14. Error Recovery & Retry Logic
**Blocker:** No timeout, retry, or failure recovery specified.
- If FileReader fails, return error or retry?
- If Canvas API crashes, fallback to what?
- If logging server is down, queue locally or drop log?
- UI timeout: If validation runs >5 seconds, show loading spinner or timeout error?
- **Action needed:** Error handling flowchart + timeout thresholds + retry policies

---

### 15. Performance Targets
**Blocker:** "~1–2 seconds" is not an SLA.
- What's the target? p50? p95? p99?
- Does 1–2s include FileReader + Canvas API + logging, or just validation logic?
- SLA for production: 95% under 2s? 99% under 3s?
- Concurrent load: Can system handle 10 concurrent validations? 100?
- **Action needed:** SLA spec (e.g., "p95 validation <2s; p99 <3s; support 100 concurrent users")

---

### 16. Concurrent Validation Handling
**Blocker:** No spec for what happens if applicant uploads while validation is running.
- Queue uploads? Cancel previous? Reject with "Wait for previous validation to complete"?
- Can UI handle multiple file inputs?
- **Action needed:** UI/UX spec for concurrent uploads

---

### 17. Browser Compatibility & Graceful Degradation
**Blocker:** "Canvas API" but doesn't specify support matrix.
- IE11 support? (Canvas API exists but some features are limited)
- Safari on iPhone: HEIC support varies by version
- If Canvas API fails, fallback to what? Skip photo validation? Reject?
- **Action needed:** Browser support matrix + fallback strategy

---

### 18. Multilingual String Completeness
**Blocker:** "All 10 check labels + pass/fail messages in 4 languages" — but current prototype has gaps.
- Where are the strings stored? HTML? JSON? Database?
- Which strings need translation? Error messages? Officer notes?
- QA: How to verify translation completeness?
- **Action needed:** String resource map (all keys + translation requirements)

---

### 19. FLAG Rate Expectations
**Blocker:** No data on expected FLAG rate.
- With simulated 70–90% pass rates, roughly 5–30% of submissions will FLAG (depends on distribution)
- Officer capacity: How many FLAGs per day? Acceptable queue depth?
- If FLAG rate is 50%, is officer queue sustainable?
- **Action needed:** Capacity planning (expected FLAG rate + officer SLA)

---

### 20. Testing Requirements
**Blocker:** No unit test, integration test, or E2E test specs.
- Unit tests: Test each check independently? Test confidence calc?
- Integration tests: Test full 10-check flow? Test with real images?
- E2E tests: Browser-based? Simulate applicant upload + validation + officer review?
- Test data: Synthetic test images with known dimensions, EXIF tags, face positions?
- **Action needed:** Test strategy + test case matrix

---

## Open Decisions (Stakeholder Input Required)

**Decisions not yet made; require stakeholder input:**

1. **How should we handle edge cases where agent is unsure?**
   - Option A: Flag for officer review (current approach) — slower for applicant, safer for accuracy
   - Option B: Block photo with "Contact ICA" message — faster but may frustrate genuine applicants
   - Option C: Allow applicant to override after warning — preserves user choice but defeats validation purpose
   - **Question for stakeholders:** What's acceptable friction level vs. safety?

2. **Should applicants see confidence scores (75%, 82%, etc.)?**
   - Option A: Show only PASS / FLAG / FAIL (current approach) — simple, no confusion
   - Option B: Show "78% confidence; photo may need another look" — transparent, educating
   - Option C: Show detailed breakdown of each check (10 items + individual scores) — most transparent
   - **Question for stakeholders:** How much transparency without creating distrust?

3. **What should happen on the 2nd or 3rd failed resubmit attempt?**
   - Option A: Keep allowing resubmits indefinitely — applicant choice preserved
   - Option B: Offer manual officer review after 3 fails — escalates to human after repeated failures
   - Option C: Escalate to support with "Contact us for assistance" — shifts load to support team
   - **Question for stakeholders:** What's the retry limit before human intervention?

4. **Should we allow applicants to skip photo validation?**
   - Option A: Validation mandatory (current approach) — ensures compliance before backend
   - Option B: Allow "I'll upload my photo at the ICA office" path — flexibility, but skips pre-validation
   - Option C: Require validation but make it one of several upload channels — hybrid approach
   - **Question for stakeholders:** Is pre-validation required for all applicants or only online?

5. **How do we handle HEIC/HEIF format for iOS users?**
   - Option A: Support natively (Canvas API handles it) — best for iPhone users
   - Option B: Reject HEIC/HEIF, ask for PNG/JPG — simplifies backend, may frustrate iOS users
   - Option C: Offer client-side conversion (HEIC → JPG) — seamless but adds JS library
   - **Question for stakeholders:** Does ICA backend support HEIC, or must we normalize?

6. **Should flag/fail feedback be prescriptive or open-ended?**
   - Option A: "Move face to center of photo" (prescriptive, current) — applicant knows exactly what to do
   - Option B: "Face not centered. Try repositioning" (suggestive) — leaves room for user interpretation
   - Option C: "Photo does not meet guidelines. Retake the photo." (minimal) — applicant must figure it out
   - **Question for stakeholders:** How much guidance should system provide vs. leaving to applicant?

7. **When should we re-test with new ML model (when it replaces simulated checks)?**
   - Current: Usability testing (Week 4) uses simulated ML
   - Future: Real ML model (Week 6+) may have different accuracy profile
   - Decision needed: Re-test with real model? Or assume simulated ≈ real?
   - **Question for stakeholders:** Is Week 4 testing sufficient, or must we re-validate with real ML?

8. **How much human oversight of outcome decisions before deployment?**
   - **Context:** Agent decides PASS/FLAG/FAIL (deterministic, rules locked). This gates whether applicant can upload.
   - **Risk:** If outcome calculation has a bug → false rejections (good applicants blocked) or false acceptances (bad photos pass).
   - **Options:**
     - **Option A: Confirm every outcome** — Human approves before result reaches applicant. Safe but defeats automation (impractical at scale).
     - **Option B: Confirm outcomes during early deployment only** — First week/month, officer manually spot-checks all PASS and FAIL decisions before they reach applicants. After proving accuracy ≥90%, switch to Auto.
     - **Option C: Rely on monitoring + rollback** — Auto all outcomes; monitor weekly false-rejection rate; if spike detected (>20% above baseline), disable agent immediately + investigate.
     - **Option D: Confirm only if confidence is low** — Auto for PASS (90%+) and FAIL (<75%); Confirm for FLAG (75–89%, already escalates to officer).
     - **Option E: A/B test during Week 4 usability test + early deployment** — Split applicants 50/50: Group A gets Auto outcomes; Group B gets Confirm outcomes. Measure false rejection rate, false acceptance rate, officer feedback, applicant satisfaction. Run for 1–2 weeks with real traffic. Flip to whichever group performs better (lower false rejections, same or better accuracy). Risk halved (only 50% affected if auto outcomes are buggy); data-driven decision; can roll back Group A if metrics deteriorate.
   - **Question for stakeholders:** Which oversight model matches your risk tolerance? (A = highest safety, lowest speed; C = highest speed, requires strong monitoring; B/D/E = hybrid; **E is data-driven + real production test**)

---

**Generated:** 19 November 2026  
**Scenario:** B  
**Data:** All synthetic, labelled as made-up  
**Status:** Draft for stakeholder review + engineering implementation gaps identified
