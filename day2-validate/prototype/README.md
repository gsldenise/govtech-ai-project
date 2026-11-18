# ICA Passport Checker Prototype

**Date:** 18 November 2026  
**Status:** Week 4 Validation Testing  
**Scenario:** B · Reduce return submissions for passport applications

## What It Does

An interactive HTML5 prototype that helps passport applicants verify what documents they need **before submitting**, reducing incomplete applications and return rates.

### 4-Step Flow

1. **Scenario Selection** — Choose application type (Renew own / Renew child / First passport)
2. **Special Circumstances Check** — Flag edge cases (Overseas / Lost / Name change) for escalation to officers
3. **Requirements Summary** — Display context-specific documents with checkmarks
4. **Application & Documents** — Combined form + drag-and-drop file upload with document examples
5. **Review & Outcome** — Show validation results and any issues detected

### Key Features

**Context-Aware Requirements**
- Different requirements for each scenario (adult renewal vs. child renewal)
- Only shows relevant documents for applicant's situation
- Scenario persists through entire flow

**Professional File Upload UI**
- Drag & drop boxes with folder icon
- File format hints (PDF, JPG, PNG • Max 8MB)
- Document-specific examples showing what's required:
  - "Passport photo: Face clearly visible, White/light background, Recent (3 months), JPG/PNG format"
  - "Birth certificate: Full name visible, Document number shown, Issue date clear"
  - "NRIC: Front and back, Clear and legible, Not expired"
  - Similar examples for all 20+ document types

**User Experience**
- Progress bar tracks completion (4 steps)
- Back buttons on every page for easy navigation
- Visual feedback: button hover states, file upload success states
- Responsive design works on mobile & desktop

**Validation Feedback**
- Shows which documents are missing or invalid
- "Go Back & Fix" option to correct issues before resubmitting
- Outcome page displays application type and status

## Files

- **full-flow.html** — Complete prototype (~23 KB, self-contained)
- **requirements-mapping.md** — ICA requirements reference (adult renewal, child renewal)
- **checker-preview.html** — Earlier version (archive)

## How to Run

1. Start the web server from `.claude/launch.json`:
   ```bash
   python3 -m http.server 8000 --directory day2-validate/prototype
   ```
2. Open browser to `http://localhost:8000/full-flow.html`

## Validation Gate Criteria

**Engagement:** ≥25% of test users complete end-to-end  
**Clarity:** ≥75% of users can describe what to prepare without looking back at the summary

**Test scenarios:** Renewal-child, Renewal-self, First-time (5–8 users)  
**Success:** If clarity ≥75%, proceed to build. If <75%, redesign and re-test.

## Tech

- Pure HTML5, CSS3, vanilla JavaScript
- No dependencies or build step
- Works offline
- ~500 lines of code

## Known Limitations

- File uploads are simulated (show success, don't actually upload)
- Document validation is randomized (for testing user behavior)
- Special circumstances redirect to officer contact flow (no submission)
- Context is locked after branching (cannot change scenario mid-flow)

## Next Steps (Week 5+)

- **Gate decision:** Measure engagement & clarity against 75% threshold
- **Build phase:** Integrate with ICA backend, real file upload, photo AI validation
- **Production:** Deploy as embedded pre-submission step in ICA portal
