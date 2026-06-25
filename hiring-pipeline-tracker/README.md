# hiring-pipeline-tracker

Hiring pipelines get messy fast. Interviewers send notes in Slack, stages get updated in someone's spreadsheet, hiring managers ask for updates at the worst times, and candidates slip through without follow-up because nobody tracked the next step.

Two modes: a webhook to log a candidate update (stage change, interviewer notes, rating, next step), and a weekly Monday morning pipeline report showing all active candidates by stage and role, what needs action this week, and an overall health assessment.

---

## What it does

**Log candidate (webhook `/log-candidate`):**
- Takes: candidate name/email, role, stage, interviewer notes, rating, source, action required, next step date, hiring manager email
- Logs to Google Sheets Pipeline tab
- If interviewer notes are provided (>50 chars): Claude summarizes strengths, concerns, recommended next step, and a 2–3 sentence summary for the hiring manager
- If hiring manager email is provided: emails them the update with the note summary
- Returns the logged entry + note summary

**Weekly pipeline report (Monday 8:30am):**
- Loads all active (non-final-stage) candidates from the Pipeline sheet
- Aggregates by stage and role
- Identifies candidates where action is required or next step date has passed
- Claude writes a pipeline health assessment: distribution, what's urgent this week, any bottlenecks
- Sends formatted email to `HIRING_LEAD_EMAIL`

---

## Stack

n8n (webhook + weekly scheduler), Google Sheets, Anthropic Claude (claude-sonnet-4-20250514), SMTP.

---

## Setup

**Pipeline sheet** columns: `candidate_id | logged_at | candidate_name | candidate_email | role | department | stage | previous_stage | rating | source | action_required | next_step_date | notes_summary`

**Env vars:** `HIRING_SHEET_ID`, `FROM_EMAIL`, `HIRING_LEAD_EMAIL`

---

## Stages

`applied` → `phone_screen` → `technical` → `take_home` → `onsite` → `offer` → `hired` / `rejected` / `withdrawn`

Candidates in `hired`, `rejected`, or `withdrawn` are excluded from the weekly pipeline report.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/log-candidate \
  -H "Content-Type: application/json" \
  -d '{
    "candidate_name": "Kenji Watanabe",
    "candidate_email": "kenji@email.com",
    "role": "Senior Backend Engineer",
    "department": "Platform",
    "stage": "onsite",
    "previous_stage": "technical",
    "rating": 4.2,
    "source": "LinkedIn Recruiter",
    "hiring_manager_email": "priya@company.com",
    "next_step_date": "2025-05-23",
    "action_required": "Reference checks before offer",
    "interviewer_notes": "Strong system design fundamentals. Designed a clean sharded database architecture for the distributed cache problem. Struggled slightly with the concurrency question but recovered well when given a hint. Cultural fit excellent — good questions about eng culture and team structure. Tom thinks strong hire. Recommendation: move to offer pending references.",
    "reply_email": "recruiting@company.com"
  }'
```

---

## Note summarizer

When interviewer notes are provided, Claude extracts:
- Strengths observed (specific, not generic)
- Concerns or gaps
- Recommended next step (advance / hold / reject / more info needed)
- 2–3 sentence summary for the hiring manager

This summary is what gets emailed to the hiring manager — not the raw notes. Keeps the signal without the noise.

---

## Limitations

- The tracker is a log, not a full ATS. It doesn't track scheduled interview times, manage job postings, or integrate with calendars. For those capabilities, connect to Greenhouse, Lever, or Ashby via their APIs.
- Stage transitions aren't enforced — you can log a candidate at "onsite" without having logged "phone_screen" first. This is intentional for flexibility but means data quality depends on consistent usage.
- The weekly report pulls from a single sheet. If you have multiple hiring leads for different departments, add a filter by `department` before aggregating to send department-specific reports.

---

## Ideas

- [ ] Slack bot: log candidate updates via Slack command rather than API call
- [ ] Offer calculator: when stage moves to "offer", trigger a compensation analysis based on role and market data
- [ ] Source analytics: track where hired candidates come from over time to optimize sourcing spend
- [ ] ATS integration: push stage updates back to Greenhouse or Lever via their APIs

---

## License

MIT.
