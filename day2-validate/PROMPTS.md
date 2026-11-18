# Prompt chain

**Scenario:** B · ICA passport applications
**Prototype today:** An interactive form that helps applicants understand passport requirements before submitting

All data is synthetic and labelled as made up.

## 1. Expand

**Prompt:** Implement a passport photo checker to ensure that the passport photo used follows ICA guidelines that will be used when a passport photo is uploaded

**Business rules:**
- Photo format needs to be png or jpeg or jpg or HEIC or HEIF
- Background of photo should be white
- Photo needs to have even brightness
- Neutral facial expression, mouth closed
- photo needs to be clear and sharp, not pixelated
- Photo should not be distorted
- Upload needs to be 400x514 pixels
- Instructions need to be localized to 4 languages: English, Malay, Tamil and Simplified Mandarin

**Edge cases:**
- Landscape style photo uploads due to how the phone stores the photo -  iPhones store images with an EXIF rotation tag rather than rotating the actual pixel data. So a photo the user sees as upright on their screen may be stored as landscape with a tag telling the renderer to rotate it.
- Face not centered well in the photo
- Multiple faces in the photo that was uploaded
- Accessibility for users that are unable to see

**Result:** _fill in after you run it: what worked, what broke._

## 2. Behavior

**Prompt:** When the photo uploaded does not meet the business rules, an error needs to be shown to the user and indicate what the error is

**Forced failure:** Upload of photo unsuccessful due to timeout issues.

**Business rules:**
- Do not allow upload if the photo is in a different format from what was specified
- Do not allow upload if the photo is of a different pixel size

**Edge cases:**
- Photo that is not a passport photo

**Result:** _fill in after you run it: what worked, what broke._

## 3. Refine

**Prompt:** On the segment for photo uploaded, provide clear instructions on the photo's requirements to increase the chance of the correct photo being uploaded the first time.

**Business rules:**
- Photo guidelines clearly shown as guidelines in the photo upload segment
- Highlight the guidelines that were not met if a photo was uploaded and did not meet the criteria
- Remove the upload that did not pass the photo checks

**Edge cases:**
- Photo that is not a passport photo

**Result:** _fill in after you run it: what worked, what broke._
