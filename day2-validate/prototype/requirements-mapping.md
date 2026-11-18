# Passport Requirements Mapping · Validated against ICA

**Date:** 18 November 2026  
**Source:** ICA official requirements documentation

## Scenario 1: Adult Passport Renewal (16+), Standard Case

### Identity Documents

| Requirement | ICA Rule | Notes |
|---|---|---|
| Singpass login | Required to access MyICA | Some personal info pre-fills |
| NRIC number | Used to identify applicant | |
| Digital colour passport photo | Recent photo within last 3 months | Must meet ICA guidelines |

### Supporting Documents

| Requirement | ICA Rule | Notes |
|---|---|---|
| Current passport number and expiry date | Used to verify eligibility | Applicant enters details in form |
| Full name, date of birth, sex | Personal particulars required by ICA | Pre-filled via Singpass where available |
| Contact details | Required by ICA | Email, phone, address |
| Reason for renewal | Eligibility check | Expiring, expired, or out of pages |

### Fees & Processing

| Item | ICA Requirement | Notes |
|---|---|---|
| Application fee | S$70 | Online application |
| Payment method | Credit card, debit card, PayNow | Supported payment methods |
| Processing time | 1–2 weeks | Approximate, in Singapore |
| Passport validity | 10 years | For citizens 16+ |

### Photo Specifications

| Requirement | Specification |
|---|---|
| Recency | Taken within last 3 months |
| Dimensions | 400 × 514 pixels |
| Formats | JPG, JPEG, PNG, HEIC, HEIF |
| Max file size | 8 MB |
| Appearance | Follow ICA facial visibility, expression, background guidelines |

### Eligibility for Standard Renewal

- Passport is expired OR expiring within 9 months OR running out of pages
- Singapore citizen aged 16+
- No changes to personal particulars (name, nationality, etc.)

---

## Scenario 2: Child Passport Renewal (Under 16), One Parent Applying

### Identity Documents

| Requirement | ICA Rule | Notes |
|---|---|---|
| Child birth certificate or citizenship certificate | Proof of identity & citizenship | Required document |
| Digital colour passport photo | Recent photo within last 3 months | Child's photo, meets ICA guidelines |
| Parent Singpass login | Required to access MyICA | Parent/guardian initiates application |

### Supporting Documents

| Requirement | ICA Rule | Notes |
|---|---|---|
| Child current passport | If any | Optional if no prior passport |
| Parent NRIC or valid passport | Required for collection | Parent/guardian document for ID verification |
| Parental consent | Required for applicants under 16 | Part of application form |
| Child personal details | Full name, date of birth, sex | Entered in application |

### Fees & Processing

| Item | ICA Requirement | Notes |
|---|---|---|
| Application fee | S$70 | Online application |
| Payment method | Credit card, debit card, PayNow | Supported payment methods |
| Processing time | 1–2 weeks | Approximate, in Singapore |
| Passport validity | 5 years | For children under 16 |
| Carry-over validity | Up to 9 months | From previous passport, if any |

### Photo Specifications

Same as adult renewal:
- Taken within last 3 months
- 400 × 514 pixels
- JPG, JPEG, PNG, HEIC, HEIF
- Max 8 MB
- Follow ICA guidelines

### Collection Requirements

| Age Group | Collection Method |
|---|---|
| 6 years and above | Child must collect in person |
| Below 6 years | Parent or guardian can collect |

---

## Key Differences Summary

| Aspect | Adult (16+) | Child (Under 16) |
|---|---|---|
| Passport validity | 10 years | 5 years |
| Application fee | $70 | $70 |
| Processing time | 1–2 weeks | 1–2 weeks |
| Photo | Digital colour, within 3 months | Digital colour, within 3 months |
| Parental consent | N/A | Required |
| Collection | Self | 6+: in person / <6: parent/guardian |
| Carry-over validity | N/A | Up to 9 months from previous |

---

## Known Return Reasons (From Opportunity Statement)

### Top 3 Return Reasons (All Scenarios)

1. **Missing supporting document (41%)** — Applicant doesn't know what's required for their situation
2. **Photo non-compliance (28%)** — Photo doesn't meet format, recency, or guideline requirements
3. **Incomplete or mismatched form fields (18%)** — Personal details don't match documents

### How the Checker Addresses These

- **Missing documents:** Tailored summary shows only relevant requirements based on applicant scenario
- **Photo issues:** Summary explicitly states photo specs (size, format, recency, guidelines)
- **Form mismatches:** Summary lists what personal details are needed (name, DOB, sex, reason)

---

## Notes for Validation Testing

- Every requirement listed in the checker output traces back to this mapping
- Photo specifications are ICA's official technical requirements
- Processing times are ICA's stated estimates
- Fees are current as of 18 November 2026
- This mapping should be reviewed if ICA requirements change
