# AI user flow: When the applicant uploads their passport photos for submission

**Scenario:** B · ICA passport applications

All data in this file is synthetic.

## The moment

When the applicant uploads their passport photos for submission

## The iceberg

### Above the line: what the user sees

Users upload the photos, gets response on whether the photo matches the applicant and follows guidelines

### Below the line

- Data: Data is pulled from various sources - Singpass for personal data, Criminal database/No fly lists etc
- Model: Facial recognition model + passport photo guidelines model
- Rules: Application needs to be filled with the correct information from Singpass, photo that is uploaded needs to be in certain formats (jpeg and png) with a size below a certain size.
- Handoffs: If the passport assessment is below a confidence rating of 85% but above 70%, if the application triggers redflags when checked against criminal databases and no fly zones. If below 70% the applicant should reupload their photo.

## The flow

1. **Trigger:** When an applicant starts an application
   - Trust gap: none
2. **AI step:** Scans the photo that is uploaded to ensure compliance with guidelines
   - Trust gap: Black box. If photo is rejected without clear reasoning as to why, users will be frustrated and will not know how to correct their photo.
3. **Human check:** When applications are approved for submission, officers will need to check that AI approved photos are within guidelines.
   - Trust gap: none
4. **Outcome:** Less photo rejections.
   - Trust gap: none

## Fallback when the AI is unsure (Advanced)

If the passport assessment is below a confidence rating of 85% but above 70%, the application should go to an officer to assess.

## How we measure trust (Expert)

Number of applications that are approved without human intervention is above 90%.
