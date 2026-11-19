# Agent Run Log: ICA Passport Photo Validator

**Date:** 19 November 2026  
**Agent:** Photo Validator (CLAUDE.md)  
**Data:** Synthetic, made-up applicants only  

---

## Run 1: Photo Passes All Checks → PASS

### Input
```json
{
  "file": {
    "name": "passport_nov2026_001.png",
    "size": 1842356,
    "type": "image/png",
    "mime_verified": true,
    "dimensions": [400, 514],
    "data": "<binary: white background, neutral face centered>"
  },
  "scenario": "renewal-self",
  "language": "en"
}
```

### Goal
Validate passport photo against ICA 10-point checklist. Return outcome (PASS/FLAG/FAIL) with confidence score and specific feedback.

### Plan
1. **Pre-flight checks:** File type, size, readability
2. **Strict checks:** Format, dimensions, background, brightness
3. **ML checks:** Expression, clarity, distortion, face-centered, single-face
4. **EXIF:** Orientation
5. **Outcome:** Compile result and log

### Act

**Step 1: Pre-flight**
- MIME type: `image/png` ✓ (supported)
- File size: 1,842 KB < 8 MB ✓
- Can read: Yes ✓
- Proceed

**Step 2: Strict Checks**
- Format: PNG ✓
- Dimensions: 400×514 ✓
- Background: Corners sampled → RGB (245, 248, 250) → all >200 ✓
- Brightness: Pixel avg luminosity = 142 → >100 ✓
- All strict checks pass

**Step 3: ML Checks** (simulated; seed = hash(filename + timestamp))
- Expression: neutral, mouth closed → 70% pass rate → **PASS** ✓
- Clarity: sharp, not pixelated → 80% pass rate → **PASS** ✓
- Distortion: no warping → 90% pass rate → **PASS** ✓
- Face-centered: position (198, 256) vs center (200, 257) → within ±30px → 75% pass rate → **PASS** ✓
- Single-face: face count = 1 → 85% pass rate → **PASS** ✓

**Step 4: EXIF**
- Rotation tag: 1 (normal) → already correct → **PASS** ✓

**Step 5: Outcome**
- All 10 checks pass
- Confidence: 100%
- Result: PASS

### Observe

**Validation complete**
```json
{
  "result": "PASS",
  "confidence": 100,
  "message_en": "✓ Photo meets all ICA requirements.",
  "feedback_en": ["Your photo is ready to submit."],
  "next_action": "proceed_to_submission",
  "officer_review": false,
  "checks": {
    "format": true,
    "dimensions": true,
    "background": true,
    "brightness": true,
    "expression": true,
    "clarity": true,
    "distortion": true,
    "face_centered": true,
    "single_face": true,
    "exif_rotation": true
  }
}
```

### Check
- ✓ All checks pass
- ✓ Confidence 100%
- ✓ Message in correct language (English)
- ✓ No PII collected or logged
- ✓ No images stored

### Log Entry
```
2026-11-20T09:15:32Z | photo_validated | ph_a1k9m3 | renewal-self | en | PASS | 100 | none | false
```

---

## Run 2: Photo Borderline → FLAG for Officer Review

### Input
```json
{
  "file": {
    "name": "passport_nov2026_002.jpg",
    "size": 2156891,
    "type": "image/jpeg",
    "mime_verified": true,
    "dimensions": [400, 514],
    "data": "<binary: white background, face slightly off-center to right>"
  },
  "scenario": "renewal-child",
  "language": "ms"
}
```

### Goal
Validate passport photo. If borderline (75–89% confidence), FLAG for officer; don't auto-reject.

### Plan
1. Pre-flight checks
2. Strict checks
3. ML checks (expect one borderline)
4. Confidence calculation
5. Decide: PASS/FLAG/FAIL
6. Log and escalate to officer if FLAG

### Act

**Step 1: Pre-flight**
- MIME type: `image/jpeg` ✓
- File size: 2,157 KB < 8 MB ✓
- Can read: Yes ✓
- Proceed

**Step 2: Strict Checks**
- Format: JPG ✓
- Dimensions: 400×514 ✓
- Background: RGB (238, 240, 242) → all >200 ✓
- Brightness: avg 138 → >100 ✓
- All strict pass

**Step 3: ML Checks** (seed = hash("passport_nov2026_002.jpg" + "2026-11-20T09:22:14Z"))
- Expression: neutral, mouth closed → PASS ✓
- Clarity: sharp → PASS ✓
- Distortion: no warp → PASS ✓
- Face-centered: position (215, 258) vs center (200, 257) → **15px right of center, within tolerance but borderline → FAIL** ✗
- Single-face: count = 1 → PASS ✓
- EXIF: normal → PASS ✓

**Step 4: Outcome Decision**
- 9/10 checks pass, 1 ML check fails (face-centered)
- Confidence: 75–89% range (depends on how far off)
- 15px is within ±30px tolerance but close to edge → set confidence to 78%
- Result: FLAG (not FAIL; officer can still approve)

### Observe

**Validation complete; escalate to officer**
```json
{
  "result": "FLAG",
  "confidence": 78,
  "message_ms": "⚠ 1 check borderline: Face well-centered in photo",
  "feedback_ms": [
    "Wajah Anda sedikit ke kanan. Usahakan untuk menempatkan wajah di tengah."
  ],
  "next_action": "officer_review",
  "officer_review": true,
  "officer_note_en": "Photo borderline on face-centering (15px right of center, within tolerance but close edge). Applicant may resubmit or officer may approve. Decision: [Officer to decide]",
  "checks": {
    "format": true,
    "dimensions": true,
    "background": true,
    "brightness": true,
    "expression": true,
    "clarity": true,
    "distortion": true,
    "face_centered": false,
    "single_face": true,
    "exif_rotation": true
  }
}
```

### Check
- ✓ Confidence 78% falls in FLAG range (75–89%)
- ✓ Feedback in Malay (language preference respected)
- ✓ Officer note created for human review
- ✓ No PII collected
- ✓ No images stored

### Log Entry
```
2026-11-20T09:22:14Z | photo_validated | ph_b7x2k5 | renewal-child | ms | FLAG | 78 | face_centered | true
```

---

## Run 3: Unsupported Format → REFUSE

### Input
```json
{
  "file": {
    "name": "my_photo.bmp",
    "size": 5242880,
    "type": "image/bmp",
    "mime_verified": true
  },
  "scenario": "first-self",
  "language": "zh"
}
```

### Goal
Reject unsupported file format early. Do not attempt to validate.

### Plan
1. Check MIME type against allowed list
2. If not in {png, jpeg, jpg, heic, heif}, reject immediately
3. Return user-friendly error in applicant's language
4. Log rejection

### Act

**Step 1: File Type Check**
- MIME type: `image/bmp`
- Allowed list: {image/png, image/jpeg, image/jpg, image/heic, image/heif}
- Match: No ✗
- **REJECT immediately. Stop. Do not read pixel data.**

**Step 2: Error Message**
- Language: Chinese (zh)
- Error key: "unsupported_format"
- Message: "不支持的文件格式。请上传 PNG、JPG、JPEG、HEIC 或 HEIF。"

### Observe

**Validation rejected at pre-flight**
```json
{
  "result": "ERROR",
  "error_code": "unsupported_format",
  "confidence": 0,
  "message_zh": "❌ 不支持的文件格式。请上传 PNG、JPG、JPEG、HEIC 或 HEIF。",
  "next_action": "user_action_required",
  "officer_review": false,
  "checks": null
}
```

### Check
- ✓ Rejected before attempting pixel analysis
- ✓ Error message in applicant's language (Chinese)
- ✓ Clear guidance on supported formats
- ✓ No image data read or stored
- ✓ No PII involved

### Log Entry
```
2026-11-20T09:28:47Z | validation_error | ph_c4m6n9 | first-self | zh | ERROR | 0 | unsupported_format | false
```

---

## Run 4 (Bonus): PII Submission Attempt → REFUSE

### Input (attempted)
```json
{
  "file": {...},
  "applicant_name": "Tan Wei Ming",
  "nric": "S1234567A",
  "scenario": "renewal-self",
  "language": "en"
}
```

### Goal
Reject any submission that includes personal identifiable information. Return error and stop.

### Plan
1. Scan input for PII fields (name, NRIC, DOB, passport number)
2. If found, reject immediately
3. Return error message
4. Log rejection without storing PII

### Act

**Step 1: Input Validation**
- Check for PII fields: applicant_name, nric, dob, passport_number
- Found: `applicant_name`, `nric` ✗
- **REJECT immediately.**

**Step 2: Error Message**
- Explain: Photo validation is anonymous
- Instruct: Submit only the photo file

### Observe

**Validation rejected; PII detected**
```json
{
  "result": "ERROR",
  "error_code": "pii_submitted",
  "confidence": 0,
  "message_en": "❌ Photo validation is anonymous. Do not submit personal details (name, NRIC, passport number). Submit only the photo file.",
  "next_action": "user_action_required",
  "officer_review": false,
  "checks": null
}
```

### Check
- ✓ Rejected before reading any photo data
- ✓ No PII stored in logs
- ✓ Clear error message explaining why

### Log Entry
```
2026-11-20T09:31:55Z | validation_error | ph_d5m7q2 | renewal-self | en | ERROR | 0 | pii_submitted | false
```

---

## Summary

| Run | Scenario | Language | Input | Result | Confidence | Action | Officer? | Log Status |
|---|---|---|---|---|---|---|---|---|
| 1 | renewal-self | en | Valid PNG, all checks pass | PASS | 100% | Proceed to submit | No | ✓ Logged |
| 2 | renewal-child | ms | JPG, face 15px off-center | FLAG | 78% | Officer review | Yes | ✓ Logged |
| 3 | first-self | zh | BMP file (unsupported) | ERROR | 0% | User resubmit format | No | ✓ Logged |
| 4 | renewal-self | en | PII submitted with photo | ERROR | 0% | User resubmit anon | No | ✓ Logged |

---

## Notes for Agent Implementation

1. **Deterministic validation for same photo:** If applicant resubmits the same file (same name + timestamp), validation result must be identical (use as seed for random checks)

2. **Officer FLAG flow:** When confidence 75–89%, create officer note but do NOT block applicant. Officer decides: approve or request resubmit.

3. **Privacy:** All logs use photo ID hash, never applicant name or NRIC.

4. **Language:** All feedback in applicant's language preference (en, ms, ta, zh).

5. **Synthetic data:** All applicants, photos, and details are made-up. No real ICA applicants or documents used.

---

**Generated:** 19 November 2026  
**Scenario:** B  
**Status:** Agent run log complete
