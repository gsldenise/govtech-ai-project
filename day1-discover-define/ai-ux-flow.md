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

## AI-UX readiness

**Scenario:** B · ICA passport applications
**Moment:** When the applicant uploads their passport photos for submission
**Level reached:** 2 of 3
**What we ship:** Stay on a validator UI until the correction rate shows the AI removes more labor than it creates.

### Levels

- Level 01 · Trigger Accuracy: Pass. AI facial verification of photo + passport photo guidelines checks will always be invoked to check if the passport photo uploaded meets the guidelines provided. the results of said verification will need to meet a certain threshold to allow the application to be submitted to the officers for final approval.

Submit application: result should show 90% confidence or higher.
Submit application with warning: result should show 75% to 89% confidence rating where an officer will see the application being flagged for additional reviews as the AI output lacks confidence.
Applicant needs to resubmit the photo: Anything below 75% needs to be resubmitted. The system will indicate what is causing the system to flag it as requiring resubmission.
- Level 02 · System Transparency: Pass. SLA required for the AI solution to return the results of photo validation. Additionally, the errors need to be in a human readable format so that the users can make the appropriate changes to their photos for resubmission.
- Level 03 · Invisible Integration: Not yet

### Trust gaps

- Black box: If photo is rejected without clear reasoning as to why, users will be frustrated and will not know how to correct their photo. Move: the errors need to be in a human readable format so that the users can make the appropriate changes to their photos for resubmission.
- Hallucination: If a well taken photo by a professional is constantly being rejected as not following guidelines. Move: Robust user testing needs to be worked into the solution for validation before utilizing it as part of the workflow.
- Control: Applicants feel that their photos follow guidelines and they have no way to override the decision made by AI to have a human determine if the photo follows the guidelines Move: Robust user testing needs to be worked into the solution for validation before utilizing it as part of the workflow. Have to decide if we want to allow overrides of AI decisions and allow manual decision - this will most likely defeat the purpose of the AI flow.
