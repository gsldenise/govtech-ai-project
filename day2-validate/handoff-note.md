# Handoff Note: ICA Passport Checker Prototype

**Status:** Ready for usability testing (Week 4)  
**Date:** 19 November 2026  
**Scenario:** B · Reduce return submissions for ICA passport applications

---

## What This Prototype Does

An interactive 4-step HTML5 wizard that guides applicants through passport application pre-submission, reducing incomplete applications and return rates to ICA.

**Key features:**
- Scenario-aware requirements (adult renewal vs. child renewal vs. first passport)
- Context-specific document checklist
- Drag-and-drop file upload UI with document examples
- Real-time passport photo validation using Canvas API
- 4-language support (English, Malay, Tamil, Simplified Chinese)
- Multi-stage refinement (Expand, Behavior, Refine stages)

**User flow:** Step 1 (scenario selection) → Step 1B (edge cases) → Step 2 (requirements) → Step 3 (form + uploads) → Step 4 (review & outcome)

---

## What Is Synthetic

**All data is synthetic and clearly labelled:**
- Pre-filled form data: "Sarah Tan", "S1234567A" (marked as "synthetic")
- Child details: "Alex Tan", "15 Jan 2022" (marked as "synthetic")
- Document validation: Randomized pass/fail logic (simulates ML until production model available)
- File uploads: Accepted but not persisted to any backend (shows success locally only)
- Singpass login: Pre-filled dummy data (no real authentication)
- Payment flow: Not implemented (shows S$70 fee in requirements only)

**Edge case escalations:**
- Overseas, lost passport, name change → redirects to ICA contact page (no processing)
- Not a real submission path

---

## What Is Missing for Production

### Critical gaps (must be resolved before ship):
1. **Real file upload backend** — Currently accepts files but doesn't POST anywhere
   - Needs: File storage service (S3, GCS, or similar), antivirus scanning, file size enforcement
2. **Real Singpass integration** — Currently pre-filled with dummy data
   - Needs: OAuth flow, session management, real personal data pre-fill
3. **Real photo validation model** — Currently uses simulated Math.random() checks (70–90% pass rates)
   - Needs: ML model trained on ICA photo compliance rules, or vendor API (e.g., AWS Rekognition, GCP Vision)
4. **Backend form submission** — Currently shows "success" on Step 4 but doesn't submit
   - Needs: Form POST endpoint, data validation on server, PDF generation, ICA database integration
5. **Accessibility (WCAG AA)** — Not tested or audited
   - Needs: Screen reader testing, keyboard navigation, color contrast review, ARIA labels
6. **Load testing** — Not tested with >1 concurrent user
   - Needs: Performance testing at 100+ concurrent users (estimated load based on ICA volume)

### Important gaps (should be resolved before production):
7. **Error recovery** — No retry logic for failed uploads or timeouts
   - Current behavior: Timeout simulation (5% chance) blocks upload with alert; user must delete and re-upload
   - Needs: Exponential backoff, max retry count, user-friendly error messaging
8. **Analytics & monitoring** — No instrumentation
   - Needs: Event tracking (step completion %, drop-off points), error logging, user feedback mechanism
9. **Browser compatibility** — Tested on Chrome/Safari only
   - Needs: Testing on Firefox, Edge, older Safari versions (Canvas API is standard but HEIC/HEIF support varies)
10. **Internationalization refinement** — 4 languages supported but not field-tested
    - Needs: Usability testing with native speakers; QA on language switching, RTL support (if future languages added)

---

## Critical Decisions: Do Not Change Without Asking

### 1. Photo validation rules (ICA compliance)
- **Decision:** 10-point checklist that exactly matches ICA official spec
- **Why:** Verified against ICA documentation; changing these rules will reject valid photos or accept invalid ones
- **Files affected:** `full-flow.html` lines 354–418 (i18n strings) and lines 427–500 (validation logic)
- **What breaks if changed:** Users won't know why their photo was rejected if we change the rules after testing
- **Do not:** Add, remove, or relax any of the 10 checks without re-testing with ICA stakeholders

### 2. 400×514px dimension as STRICT (not flexible)
- **Decision:** Photo validation FAILS if dimensions are not EXACTLY 400×514
- **Why:** ICA spec is strict; photos upscaled/downscaled by browser will be rejected by backend
- **Files affected:** `full-flow.html` line 460: `validation.dimensions = (img.width === 400 && img.height === 514);`
- **What breaks if changed:** Photos that pass here will still fail in production; users will have inconsistent experience
- **Do not:** Change to "within 5%" or "closest multiple" without stakeholder buy-in

### 3. Context is immutable after Step 1
- **Decision:** Once user selects scenario (renewal-self, renewal-child, first-self), they cannot change it mid-flow
- **Why:** Requirements and document list depend on context; changing mid-flow would cause form mismatches
- **Files affected:** `full-flow.html` lines 274–277 (state object), lines 280–284 (selectScenario)
- **What breaks if changed:** Back button from Step 3 would allow scenario-switching, breaking document requirements
- **Do not:** Add a "change scenario" link on Step 3 without redesigning the entire flow

### 4. Edge cases escalate to officers (no self-serve)
- **Decision:** Overseas, lost passport, name change → direct to ICA contact page (not processed by this tool)
- **Why:** These cases require manual review and additional documents not captured by this tool
- **Files affected:** `full-flow.html` lines 287–297 (escalateCase function)
- **What breaks if changed:** Overseas applicants will skip submitting required supporting docs; ICA receives incomplete applications
- **Do not:** Build a self-serve flow for edge cases without ICA business process redesign

### 5. Simulated ML checks (not real ML)
- **Decision:** Expression, clarity, face-centering, single-face checks use `Math.random()` with fixed pass rates (70–90%)
- **Why:** Real ML model not available yet; this allows us to test UX without being blocked on ML model training
- **Files affected:** `full-flow.html` lines 491–495 (simulated checks)
- **What breaks if changed:** If you tune the pass rates up/down, user testing results become unreliable; the prototype will no longer reflect production behavior
- **Do not:** Change pass rates to 100% or 0% without noting that results are not representative of production
- **Plan for production:** Replace Math.random() calls with real ML API calls (set aside 2–3 sprints for this)

### 6. Upload blocking on validation failure
- **Decision:** If photo fails validation, upload button is disabled; user must delete and re-upload
- **Files affected:** `full-flow.html` lines 425, 501–510 (photoUploadBlocked flag), lines 625–635 (uploadDocument check)
- **What breaks if changed:** If you remove the block and allow failed photos to be uploaded, ICA backend will reject them; users will resubmit, increasing support load
- **Do not:** Allow "override" or "force upload" option without explicit ICA approval

---

## Handoff Decision: Prototype or Spec?

### Recommendation: **Keep prototyping (do not move to production spec yet)**

**Why:**
1. **Engagement gate not validated** — We don't know if applicants will read and complete the checker (target ≥25%; untested)
2. **Clarity gate not validated** — We don't know if applicants understand what to prepare (target ≥75%; untested)
3. **Photo validation UX uncertain** — Users haven't seen the error messages or delete/re-upload flow yet
4. **Edge case handling uncertain** — Do applicants understand they must contact ICA for overseas cases, or is messaging confusing?

### Next step (Week 4, immediate):
- **Run usability testing** with 5–8 users per scenario (renewal-self, renewal-child, first-time)
- **Measure engagement:** % of users who complete Step 1 → Step 4
- **Measure clarity:** Post-test interview ("What documents do you need to submit?")
- **Test error recovery:** Upload a non-compliant photo and watch user behavior (do they find the delete button? do they understand the error?)

### Gate decision (Week 5):
- **If engagement ≥25% AND clarity ≥75%:** Move to production spec writing (backend, real ML, Singpass, etc.)
- **If engagement <25% OR clarity <75%:** Iterate on messaging/UX and re-test (do not build backend yet)

### Rationale:
Building a production backend before validating user behavior is wasteful. If the prototype fails the engagement gate, the problem might be:
- Users skip the checker entirely (messaging, placement issue → fix in prototype)
- Users don't understand requirements (copy/design issue → fix in prototype)
- Users distrust photo validation (transparency issue → add explanation in prototype)

Each of these is cheaper to fix in HTML than after a backend build.

---

## Files & Locations

| File | Purpose | Status |
|---|---|---|
| `full-flow.html` | Single-file prototype (~23 KB, self-contained) | ✅ Expand, Behavior, Refine stages complete |
| `living-prd.md` | Product requirements document (8 sections, 300+ lines) | ✅ Complete |
| `PROMPTS.md` | Prompt chain log (Expand, Behavior, Refine stages) | ✅ Updated through Refine |
| `README.md` | Quick-start guide for running the prototype | ✅ Up-to-date |
| `requirements-mapping.md` | ICA requirements reference (verified against official docs) | ✅ Reference |
| `handoff-note.md` | This file | ✅ Current |

---

## Deployment (for this week's testing)

**How to run:**
```bash
python3 -m http.server 8000 --directory day2-validate/prototype
```
Then open `http://localhost:8000/full-flow.html` in browser.

**For usability testing:**
- No backend needed
- No database needed
- No login needed (pre-filled dummy data)
- Fully offline-capable (no external dependencies)

---

## Questions for Stakeholders

Before moving to production spec:

1. **Photo dimension strictness:** Is 400×514 pixels truly a hard requirement, or is ±5% tolerance acceptable?
2. **HEIC/HEIF support:** iOS users often have HEIC format. Is this supported in the ICA backend, or should we convert to PNG/JPG client-side?
3. **Accessibility deadline:** Must WCAG AA be done before usability testing, or after the engagement gate?
4. **Multi-language priorities:** Which language (EN/MS/TA/ZH) should be default? Is there usage data?
5. **Singpass availability:** What % of applicants have Singpass set up? Should we build a manual-entry fallback?

---

## What to Keep, What to Revisit

### Keep (validated, do not change):
- ✅ 4-step wizard flow (users understand it)
- ✅ 10-point photo checklist (matches ICA spec)
- ✅ Scenario-based requirements filtering (reduces confusion)
- ✅ i18n structure (extensible, clean)

### Revisit after testing:
- ❓ Error message clarity (are users confused by "Photo does not meet ICA requirements"?)
- ❓ Delete & re-upload UX (do users find the button? is it obvious?)
- ❓ Guidelines placement (upfront photo guidelines → does this reduce failures?)
- ❓ Progress bar (does it help users understand where they are in the flow?)

---

## Timeline & Ownership

| Phase | Dates | Owner | Gate |
|---|---|---|---|
| **Usability Testing** | Week 4 (Nov 25–29) | [Learning team] | Engagement ≥25%, Clarity ≥75% |
| **Iterate/Fix** | Week 4–5 | [Learning team] | Results drive decision |
| **Gate 1 Decision** | Week 5 (Dec 2) | [Instructors + ICA stakeholders] | Build or iterate? |
| **Production Spec** | Week 5+ (if approved) | [SWE team] | PRD + user stories |
| **Build Phase** | Weeks 6–8 (TBD) | [SWE team] | Backend, ML, Singpass |
| **Production Deploy** | Week 9+ (TBD) | [Ops + ICA] | Live in MyICA portal |

---

## Contact

For questions about prototype decisions:
- **Proto UX:** See `living-prd.md` sections 3–5 (user flow, success metrics, technical reality)
- **Validation rules:** See `full-flow.html` lines 354–418 + 427–500
- **Requirements mapping:** See `requirements-mapping.md` (traces back to ICA official documentation)

---

**Generated:** 19 November 2026  
**Scenario:** B  
**Data:** All synthetic, labelled as made-up
