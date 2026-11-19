# Living PRD: ICA Passport Checker Prototype
**Date:** 19 November 2026  
**Status:** Week 4 Validation Testing (Expand Stage Complete, Behavior & Refine Pending)  
**Scenario:** B · Reduce return submissions for ICA passport applications  
**Data:** Synthetic, labelled as made-up

---

## 1. Product Overview

**What it does:** An interactive HTML5 pre-submission wizard that helps passport applicants verify what documents they need *before* submitting, reducing incomplete applications and return rates to ICA.

**Why it exists:** ICA receives 41% returns due to missing documents, 28% due to photo non-compliance, and 18% due to incomplete forms. This prototype runs before submission to catch gaps early.

**Where it lives:** 
- Single self-contained HTML file: `day2-validate/prototype/full-flow.html` (~23 KB)
- Runs in any browser; no build step, no dependencies
- Deployed as embedded pre-submission step in ICA portal (intended)

**Key user outcomes:**
- Applicant knows exactly what documents to prepare before starting
- Passport photo meets ICA guidelines before upload (validated client-side)
- Personal details pre-filled where available (Singpass integration)
- Applicant sees real-time feedback on what passed/failed

---

## 2. Problem & Riskiest Assumption

**The problem:**
- Applicants don't know what documents ICA requires for their specific scenario (adult renewal vs. child renewal varies significantly)
- 28% of returns are photo non-compliance — applicants submit wrong dimensions, wrong background, wrong format
- Applicants fill forms incorrectly because they don't know whether to enter their own details or their child's
- No pre-submission validation means ICA staff process incomplete applications, causing delays

**Riskiest assumption (blocks gate decision):**
- **Assumption:** Applicants will read and understand a pre-submission requirements summary and won't skip it
- **Risk:** If engagement is <25%, applicants bypass the checker and returns stay at 41%
- **How we test this:** Measure % of test users who complete end-to-end (gate: ≥25%)

**Secondary risky assumptions:**
1. Applicants trust client-side photo validation (they might think it's a trick to reject their upload)
2. Singpass login is available and pre-fills data (some applicants may not have Singpass set up)
3. Four languages (EN, MS, TA, ZH) cover 95%+ of applicants (may be incomplete for other dialects)

---

## 3. User Flow & Screen Map

**4-Step Wizard Flow:**

```
┌─────────────────────────────────────────────┐
│ STEP 1: Application Scenario Selection      │
├─────────────────────────────────────────────┤
│ • Renewing my own passport                  │
│ • Renewing my child's passport              │
│ • Applying for my first passport            │
│ [Context locked after selection]            │
└─────────────────┬───────────────────────────┘
                  │ selectScenario(type, applicant)
                  ↓
┌─────────────────────────────────────────────┐
│ STEP 1B: Special Circumstances Flag         │
├─────────────────────────────────────────────┤
│ • None of these apply (standard)            │
│ • Applying from overseas                    │
│ • Passport lost or stolen                   │
│ • Name change                               │
│ [Edge cases → escalation to officers]       │
└─────────────────┬───────────────────────────┘
                  │ proceedToRequirements(edge)
                  ↓
┌─────────────────────────────────────────────┐
│ STEP 2: Requirements Summary (Read-only)    │
├─────────────────────────────────────────────┤
│ • Context-specific checklist (6-12 items)   │
│ • Examples of what each document includes   │
│ • Scenario name displayed (e.g. "Renew     │
│   your own passport - Standard Case")       │
└─────────────────┬───────────────────────────┘
                  │ goToStep(3)
                  ↓
┌─────────────────────────────────────────────┐
│ STEP 3: Combined Form + Document Upload     │
├─────────────────────────────────────────────┤
│ Form Section:                               │
│ • Your name (pre-filled: "Sarah Tan")       │
│ • Your NRIC (pre-filled: "S1234567A")       │
│ • Singpass login confirmation               │
│ • Reason for renewal / Contact details      │
│                                             │
│ Document Section (per scenario):            │
│ • Each required document has:                │
│   - Document name & type                    │
│   - "Must include" requirements             │
│   - Drag & drop upload box                  │
│   - Upload button                           │
│ • PHOTO VALIDATION (NEW):                   │
│   - On upload: analyze photo vs ICA spec    │
│   - Display: ✓ or ✗ for each check         │
│   - Language switcher (EN/MS/TA/中文)      │
│   - Photo preview                           │
│   - 10 validation checks displayed          │
└─────────────────┬───────────────────────────┘
                  │ validateDocuments()
                  ↓
          [If validation fails]
                  │ goToStep(3) [retry]
                  │
          [If all pass]
                  │
                  ↓
┌─────────────────────────────────────────────┐
│ STEP 4: Review & Outcome                    │
├─────────────────────────────────────────────┤
│ Success state:                              │
│ • ✓ All documents validated                 │
│ • Scenario name & submitted details shown   │
│ • Next step: Submit to ICA portal           │
│                                             │
│ Failure state:                              │
│ • ✗ Missing or invalid documents            │
│ • Which docs failed (with reason)           │
│ • "Go Back & Fix" button                    │
└─────────────────────────────────────────────┘
```

**Branch: Escalation Path** (from Step 1B if edge case selected)
- Displays officer contact info + expected processing time
- Does not proceed to Step 3 (officer handles manually)

**State Management:**
- `step`: Current step (1, 1b, 2, 3, 4, escalation)
- `state.type`: renewal | first-time
- `state.applicant`: self | child
- `state.context`: Locked after Step 1 (prevents mid-flow scenario changes)
- `state.edgeCase`: none | overseas | lost | namechange

---

## 4. Success Metrics

**Engagement Gate (Week 4 decision point):**
- **Engagement target:** ≥25% of test users complete end-to-end (Step 1 → Step 4)
- **Current measurement:** Manual observation during usability tests (5–8 users per scenario)
- **What this means:** If <25%, applicants skip the checker; redesign needed before Gate 2

**Clarity Gate (Ship/No-Ship decision):**
- **Clarity target:** ≥75% of users can describe what to prepare without re-reading the summary
- **Measurement:** Post-test interview (open-ended question: "What documents do you need to submit?")
- **What this means:** If ≥75%, proceed to build. If <75%, iterate on requirements messaging and re-test

**Effectiveness metrics (post-launch, not measured this week):**
- Return rate (baseline 41% → target <30%)
- Photo rejection rate (baseline 28% → target <15%)
- Form completion time (baseline unknown → target <5 min)

---

## 5. Technical Reality

**What's built:**
1. **Core framework:** Vanilla HTML5 + CSS3 + JavaScript (no dependencies)
   - ~500 lines of client-side logic
   - Single HTML file = easy to embed as iframe

2. **4-Step wizard with context-aware branching**
   - All steps on separate divs; `renderStep()` swaps visibility
   - State object tracks user choices (immutable after Step 1 selection)
   - Back buttons work throughout; context never changes mid-flow

3. **Context-specific requirements filtering**
   - 6 hardcoded document sets (renewal-self, renewal-child, first-self, etc.)
   - Requirements pulled from `requirements-mapping.md` (ICA reference)
   - Document examples inline (no server calls)

4. **Drag-and-drop file upload UI**
   - File input triggers on `.file-upload-box` click
   - Shows filename + size on successful selection (e.g., "✓ passport-photo.jpg (45 KB)")
   - Visual feedback: box border turns green, background lightens
   - No actual file upload (simulated; for MVP testing only)

5. **Passport photo validation (Expand stage - LIVE)**
   - **Trigger:** When user selects a file in the "Digital passport photo" field
   - **Canvas API analysis:**
     - Reads image via FileReader
     - Measures dimensions (target: 400×514px)
     - Analyzes pixel data: background color (white %), brightness uniformity
     - Detects EXIF rotation tag
   - **Simulated ML checks** (until real ML model available):
     - Expression: neutral / mouth closed (70% pass rate)
     - Clarity: sharp / not pixelated (80% pass rate)
     - Distortion: unwarped (90% pass rate)
     - Face centered: ±30px tolerance (75% pass rate)
     - Single face: detects >1 face (85% pass rate)
   - **Output:** Pass/fail for each of 10 checks; overall ✓ or ✗
   - **UI:**
     - Language selector buttons (EN, MS, TA, 中文) above results
     - Each language has full i18n strings for all 10 checks
     - ✓ or ✗ icon + color (green/red)
     - Photo preview embedded
     - Validation result shown inline

6. **i18n (Internationalization)**
   - 4 complete language sets (en, ms, ta, zh)
   - All validation check labels translated
   - Pass/fail messages in each language
   - Language switching triggers re-render of validation panel

7. **Responsive design**
   - Max-width 600px container (fits mobile + tablet)
   - Touch-friendly buttons (min 44px tap target)
   - Progress bar (animated fill on step transitions)
   - Gradient background (works offline)

**What's NOT built (out of scope for MVP):**
- Real file upload (no backend integration)
- Real Singpass login (pre-filled with dummy data)
- Real photo ML validation (simulated with Math.random())
- Persistent storage (no database)
- Authentication / session management
- Accessibility (WCAG compliance not tested)
- Mobile app (web only)

---

## 6. Assumptions & Risks

| Assumption | Risk | Mitigation | Status |
|---|---|---|---|
| Applicants read requirements summary | If engagement <25%, checker is bypassed | Test with 5–8 real users; measure completion % | TESTING (Week 4) |
| Client-side photo validation is trusted | Applicants might think it's rigged to reject them | Show transparent rules (10 checks displayed) | MITIGATED (Expand) |
| Four languages cover user base | Some applicants speak other languages (e.g., Mandarin vs Cantonese) | Defer to GA data; add languages if needed | ACCEPTABLE (MVP) |
| Singpass integration available | Some applicants skip Singpass; form fields empty | Allow manual entry; pre-fill optional | DESIGN (Step 3) |
| Edge case escalation is acceptable | Officers overwhelmed with overseas/lost passport cases | Manual escalation is existing ICA process | LOW RISK |
| 400×514px dimension is correct | ICA updates dimension spec; old checker rejects valid photos | Version the spec in comments; re-deploy if changed | ACCEPTABLE |
| Canvas API works in all browsers | Old browsers (IE11) don't support Canvas | Degrade gracefully: skip photo validation if unsupported | FUTURE |
| Form fields are never submitted to real endpoint | Accidental data leakage if misconfigured | Data is synthetic; no real PII in MVP | LOW RISK |

---

## 7. In Scope & Out of Scope

### **In Scope (MVP, Week 4)**
- ✅ 4-step wizard for 3 scenarios (adult renewal, child renewal, first-time)
- ✅ Edge case escalation (overseas, lost, name change)
- ✅ Context-aware document requirements (6 sets)
- ✅ Drag-and-drop file upload UI with examples
- ✅ Passport photo validation (Canvas API + simulated ML)
- ✅ 4-language support (EN, MS, TA, ZH) for photo validation
- ✅ Form pre-fill (dummy data)
- ✅ Progress bar + back buttons
- ✅ Responsive design (mobile + desktop)
- ✅ Usability testing with 5–8 users per scenario

### **Out of Scope (Future Phases)**
- ❌ Real file upload / backend integration
- ❌ Real Singpass login / session management
- ❌ Real photo ML model (simulated only)
- ❌ Database persistence
- ❌ Authentication
- ❌ WCAG accessibility testing
- ❌ Native mobile apps
- ❌ Admin console / analytics dashboard
- ❌ A/B testing framework
- ❌ Feedback / survey integration

---

## 8. Engineering Recommendation

### **Recommendation: MERGE & TEST**
This prototype is ready for Week 4 usability testing. It accurately reflects ICA requirements and demonstrates the core value prop (pre-submission validation). 

### **Why it's ready:**
1. **Functionally complete:** All 4 steps work; context flows correctly; no crashes observed
2. **Reflects real requirements:** Requirements pulled from ICA documentation; examples are realistic
3. **Photo validation is defensible:** 10-point checklist matches ICA spec; simulated ML is transparent
4. **User-testable:** Low friction — no login, no file upload needed, quick to iterate based on feedback
5. **Technical debt is acceptable:** Simulated checks (not ML), no backend — this is expected for a prototype

### **Before moving to Gate 2 (build phase):**
1. **Confirm engagement ≥25%** via usability test (this week)
2. **Confirm clarity ≥75%** via post-test interview (this week)
3. **Resolve riskiest assumption:** If engagement <25%, interview users on why they didn't complete; redesign messaging
4. **Translate feedback into Phase 2 PRD:** Real Singpass, real file upload, real ML photo validation, backend storage

### **What happens in Behavior stage (next):**
- Add error states when photo validation fails
- Prevent upload of non-compliant photos
- Show "Why this photo failed" + remediation steps
- Test with actual ICA photo guidelines (edge cases: EXIF rotation, face centering)

### **What happens in Refine stage (after Behavior):**
- Move photo guidelines upfront (before upload)
- Highlight which specific guidelines failed (not just "Photo does not meet requirements")
- Let users delete and re-upload failed photos
- Test remediation flow with users

### **Ship criteria (Week 5+):**
- Gate 1: Engagement ≥25% ✓ (or iterate)
- Gate 1: Clarity ≥75% ✓ (or iterate)
- Behavior & Refine stages complete
- Accessibility audit (WCAG AA)
- Load testing (concurrent users)
- ICA stakeholder sign-off on messaging & requirements

---

## Timeline

| Phase | Week | Gate | Deliverable |
|---|---|---|---|
| **Expand** | 4 | 1 | Photo validation + 4-language support (COMPLETE) |
| **Behavior** | 4–5 | — | Error handling when validation fails (NEXT) |
| **Refine** | 5 | — | Clear guidelines + remediation UI (NEXT) |
| **Usability Testing** | 4 | 1 | 5–8 users; measure engagement & clarity |
| **Gate 2 Decision** | 5 | 1 (repeat) | Build or iterate? |
| **Build Phase** | 6+ | 2 | Backend integration, real Singpass, real ML |
| **Production** | 8+ | 3 | Deploy as embedded pre-submission step |

