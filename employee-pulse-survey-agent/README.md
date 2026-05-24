# employee-pulse-survey-agent

Most team health monitoring falls into one of two failure modes: annual engagement surveys that take an hour and produce a PDF nobody reads, or informal vibes-based management where problems only surface when someone resigns.

This runs every two weeks. Claude writes a fresh 3-question survey each time — the questions rotate through different topics (energy, clarity, progress, collaboration, work-life balance, obstacles, appreciation) so it doesn't feel like the same form on repeat. It emails each team member individually, collects responses through a webhook, anonymizes everything, tracks energy score trends across surveys, and emails the manager a plain-language analysis after each response comes in.

The manager always knows the current state of the team without invading individual privacy.

---

## What it does

**Survey send (bi-weekly, Monday 9am):**
- Loads team list from Google Sheets
- Claude writes a fresh 3-question survey email (rotates topics each cycle)
- Sends individually to each team member
- Logs the survey send to Google Sheets

**Response collection (webhook `/pulse-response`):**
- Accepts response: energy score (1–5), up to 2 open responses, optional additional score
- Saves to Google Sheets anonymously (no individual attribution in the analysis)
- Loads all responses for the current survey
- Computes: average energy score, low energy count, trend across last 4 surveys
- Claude synthesizes open responses into themes — anonymously
- Emails manager after every response: running stats + analysis
- Returns "thank you" confirmation to the respondent

---

## Stack

- **n8n** — bi-weekly scheduler + webhook
- **Google Sheets** — team list + survey log + responses
- **Anthropic Claude** (claude-sonnet-4-20250514) — survey writing + response analysis
- **SMTP** — survey emails + manager updates

---

## Setup

### 1. Create the Google Sheet

Three tabs:

**Tab: Team** — columns:
```
name | email | team | role
```
List every team member you want to include.

**Tab: Survey Log** — columns:
```
survey_id | survey_date | recipients | sent_at
```

**Tab: Responses** — columns:
```
submitted_at | survey_id | respondent_name | energy_score | open_response_1 | open_response_2 | additional_score
```

### 2. Environment variables

```
PULSE_SHEET_ID=your_google_sheet_id
PULSE_FORM_URL=https://your-form-url.com/pulse
FROM_EMAIL=pulse@yourcompany.com
MANAGER_EMAIL=manager@yourcompany.com
```

### 3. Credentials

- **Google Sheets OAuth2**
- **Anthropic API** (LangChain node)
- **Slack API** (if adding Slack delivery)
- **SMTP**

### 4. Import and activate

Import `workflow.json`, activate. Test by running the scheduler manually.

---

## The survey email

Every two weeks Claude generates a new survey email with 3 questions. The questions rotate — one is always a 1–5 numeric scale, one is always open-ended. Topics cycle through: energy and wellbeing, clarity of priorities, sense of progress, team dynamics, personal growth, work-life balance, obstacles, what's going well.

The email is kept short (under 150 words) and ends with the response form link. It's sent individually to each team member, not as a group email.

---

## Collecting responses

The `/pulse-response` webhook receives form submissions. You need a simple form that maps to these fields:

```
survey_id         → hidden field (populated from URL parameter)
respondent_name   → text field (required — needed to avoid duplicate responses)
respondent_email  → email field (optional, not used in analysis)
energy_score      → 1-5 scale (required)
open_response_1   → long text
open_response_2   → long text
additional_score  → 1-5 scale (optional second numeric question)
```

Build this with Tally.so, Typeform, or a simple HTML form. Pass `survey_id` as a URL parameter so responses link back to the correct survey.

---

## Anonymity

The responses sheet stores respondent names (for deduplication), but the Claude analysis never attributes comments to individuals. It synthesizes themes from the open responses and reports aggregate scores only. The manager email says "3 of 8 responses mentioned feeling unclear on priorities" — not who said it.

The manager can look at the raw sheet if needed, but the automated analysis stays anonymous.

---

## Manager updates

After each response comes in, the manager receives an email with:
- Current response count and average energy score
- How many low energy responses (1–2)
- Trend across the last 4 surveys
- Claude's synthesis of open response themes
- Any flag if something looks concerning

This means the manager has a real-time picture as responses arrive rather than waiting for a weekly summary.

---

## Survey frequency

Default is bi-weekly (every 2 weeks on Monday). Change in the **Bi-Weekly Monday 9am** scheduler node:
- For weekly: `weeksInterval: 1`
- For monthly: switch to `months` interval
- For a different day: change `triggerAtDay` (0=Sunday, 1=Monday, etc.)

---

## Changing the survey questions

The questions are generated fresh by Claude each cycle — you don't control them directly. If you want to lock specific questions in, edit the system prompt in the **Claude Survey Writer** node. You can add: "Always include this question: [your question]" and it will appear in every survey.

---

## Limitations

- Respondent names are stored for deduplication. If strict anonymity from even the sheet admin is required, remove the `respondent_name` field — but you'll lose the ability to detect duplicates.
- The energy score is the primary quantitative signal. If you want more data dimensions (workload, clarity, team dynamics each scored separately), add fields to the form and response schema.
- The bi-weekly send hits every person on the Team sheet. If you want to exclude someone temporarily, set an `active` column to FALSE and filter in the Load Team List step.

---

## Ideas

- [ ] Slack delivery: send the survey as a Slack DM instead of email for teams that live in Slack
- [ ] Department segmentation: separate analyses per team or department when the org grows
- [ ] Trend alerts: if average energy drops more than 1 point between surveys, send an immediate alert
- [ ] Monthly trend report: once-a-month summary of the last 2 surveys with charts

---

## License

MIT.
