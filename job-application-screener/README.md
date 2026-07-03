# job-application-screener

Initial resume screens are time-consuming, inconsistent, and depend heavily on how much attention the screener is paying. Different people screen the same resume differently, must-have criteria get applied loosely, and strong candidates with non-linear backgrounds get missed while pattern-matched resumes get through. This applies your exact criteria consistently to every application, notes the specific evidence or gap per criterion, and logs results to a Sheet for the hiring manager.

---

## What it does

Takes job title, job description, must-have criteria, nice-to-have criteria, and applicant resume/summary. Claude screens and returns:
- Overall recommendation (strong_yes/yes/maybe/no/strong_no)
- Fit score (1–10)
- Must-have assessment: each criterion with met/not-met, specific evidence or gap
- Nice-to-have assessment
- Strengths (based only on what's in the resume — no invented capabilities)
- Concerns and gaps
- 3–4 interview questions targeting gaps or verifying claims
- 2–3 sentence summary for the hiring manager
- Flags for human review (career gaps, unusual background, overqualification signals)

Logs to Google Sheets and emails the hiring manager.

---

## Stack

n8n, Google Sheets, Anthropic Claude (claude-sonnet-4-20250514), SMTP.

---

## Setup

**Sheet "Applications"** columns: `application_id | screened_at | job_title | applicant_name | applicant_email | overall_recommendation | fit_score | must_haves_met | summary`

**Env vars:** `SCREENING_SHEET_ID`, `FROM_EMAIL`

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/screen-application \
  -H "Content-Type: application/json" \
  -d '{
    "job_title": "Senior Product Manager",
    "application_id": "APP-2025-114",
    "applicant_name": "Jordan Adesanya",
    "applicant_email": "jordan@email.com",
    "hiring_manager_email": "priya@company.com",
    "must_have_criteria": [
      "3+ years PM experience at a B2B SaaS company",
      "Experience owning a product from 0 to launch",
      "Comfortable working directly with engineering without a BA layer"
    ],
    "nice_to_have_criteria": [
      "Experience with data or analytics products",
      "Have managed a team of PMs"
    ],
    "job_description": "We are looking for a Senior PM to own our core workflow product. You will work directly with engineering leads, talk to customers weekly, and be accountable for feature adoption metrics. No project manager layer — you write specs, manage the backlog, and ship.",
    "application_note": "I have been following Flowdesk since the early days and think there is a big gap in the market for lightweight ops tooling. I previously built the task management product at Beacon before it was acquired.",
    "resume_or_summary": "Jordan Adesanya — Senior PM\\n\\nBeacon Logistics (acquired by FleetCore, 2021-2024): Led product for the ops coordination suite. Shipped task management module from scratch — 0 to 2,400 DAU in 6 months. Wrote all specs, ran weekly customer calls, managed backlog directly with 4 engineers.\\n\\nWorkflow.io (2019-2021): PM on the API platform team. Owned developer onboarding flow.\\n\\nEducation: BSc Computer Science, University of Lagos, 2019.\\n\\nTools: Linear, Notion, Mixpanel, SQL (basic)"
  }'
```

**Required:** `job_title`, `job_description`, `applicant_name`, `resume_or_summary`

---

## Criteria specificity matters

"Strong communication skills" as a must-have produces a weak screen — Claude can't assess that from a resume. Specific criteria produce useful results: "3+ years PM at a B2B SaaS company", "experience writing technical specs without a BA", "shipped a product from 0 to launch". The more specific, the more the screen tells you.

---

## No fabrication

Claude is explicitly instructed not to invent experience not present in the resume. If a must-have criterion isn't addressed in the resume, it's marked not-met, not assumed present. The evidence field shows the specific text from the resume that supports each met criterion.

---

## Flags for human review

The `flags_for_human_review` field captures things that should be assessed by a person: unexplained gaps, a background that's unusual but not disqualifying, potential overqualification (could leave quickly), or claims that need verification ("acquired startup" with no further detail).

---

## Limitations

- This screens resume text, not the whole person. Non-traditional backgrounds may be systematically under-scored if the criteria are written to match conventional career paths. Review `flags_for_human_review` carefully.
- Quality depends on what's in the resume. Sparse resumes produce low-confidence screenings. If candidates submit minimal info, consider adding a structured application form.

---

## Ideas

- [ ] ATS webhook: trigger automatically when a new application is received from Greenhouse, Lever, or Workable
- [ ] Batch screening: loop through a Sheet of applications and screen all at once
- [ ] Scorecard comparison: after screening multiple candidates, generate a side-by-side comparison table
- [ ] Phone screen prep: given the screening output, generate a structured 20-minute phone screen guide

---

## License

MIT.
