# Opportunity: Reduce the number of return submissions

**Scenario:** B · ICA passport applications

All quotes and metrics are synthetic and labelled as made up.

## Problem Framework

### 1. Goal

Increase the proportion of passport applications approved on first submission, so applicants get their passports faster and ICA officers spend less time processing returns.

### 2. Problem

Applicants submit incomplete or incorrect applications because they do not know what a complete application looks like for their specific situation. The gap is not awareness that guidance exists, it is that generic guidance does not translate into what a particular applicant needs to do. The result is a 17% return rate, each return adding six days to the applicant's wait.

### 3. Context

- Who: ~520,000 applicants a year submitting online; ICA officers who process and return submissions.
- Where: The online passport application and renewal channel, currently run by a vendor withdrawing in 10 weeks.
- When: At the point of submission, after the applicant has filled in the form but before an officer reviews it. Returns happen within the officer review stage.

### 4. Constraints

- The vendor withdrawal date of 10 weeks does not move. Whatever replaces the channel must be ready within that window.
- Final approval and rejection decisions stay with ICA officers. The system cannot make or imply eligibility decisions.
- Applications contain identity documents and sensitive personal data. Any solution must handle these within ICA's data governance requirements.
- The system must clearly separate what it checks (completeness, document format) from what an officer decides (eligibility, special circumstances).
- The system must remain accurate as ICA rules change, without requiring a full rebuild each time.

### 5. Success

- The return rate drops measurably within a defined period after the new channel launches.
- The drop is attributable to fewer completeness-related returns, not fewer applications overall.
- Applicants in the highest-confusion group (parents applying for a child) show a disproportionate improvement.
- Officers report fewer returns for reasons the applicant could have self-corrected before submitting.

### 6. Explore

- Whether applicants will engage with a situation-specific check at submission time, given that 62% currently skip written guidance entirely.
- Which return reasons are most common and most preventable at the point of submission versus genuinely requiring officer judgment.
- Whether parents applying for a child understand they need to provide both the child's documents and their own.
- Whether the way return reasons are currently communicated to applicants is specific enough for applicants to fix and resubmit in one attempt.

## Evidence (synthetic)

### User quotes

- "The form just said 'attach supporting documents'. I didn't know I needed both my own IC and my child's birth certificate. I thought one would do."
- "They returned it after 10 days. The letter said 'incomplete documentation'. I had no idea what was missing — I had to call the counter to find out."
- "I'd already booked the photos and taken half a day off work. When it came back I just felt like I'd wasted everyone's time, including mine."

### Metrics

- Each return adds approximately 6 days to the applicant's total wait time.
- Only 38% of applicants read the guidance before starting their application.
- Parents applying for a child account for an estimated 34% of all returns, despite making up roughly 20% of total applications.
- The top three return reasons are: missing supporting document (41%), photo non-compliance (28%), incomplete or mismatched form fields (18%).

## Hypothesis

I am testing whether giving applicants a situation-specific completeness check at the point of submission — one that identifies their scenario and flags only the gaps relevant to them — will reduce the return rate, because most returns are caused by applicants not knowing what their specific situation requires, not by applicants being unwilling to comply.

- Risk type: Feasibility (build a functional app first)
- Kill switch: Applicants will engage with a pre-submission check when they currently skip written guidance.

If applicants treat the checker as one more piece of content to click past, there is no mechanism to catch errors before submission and the return rate will not move. No amount of checker quality compensates for zero engagement.
- Kill-switch signal: Fewer than 25% of applicants who reach the submission page complete the checker in the first four weeks after launch.
- Success signal: The return rate falls from 17% to below 10% within three months of the new channel going live, with the reduction concentrated in completeness-related returns rather than officer-discretion returns.
