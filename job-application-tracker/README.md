# job-application-tracker

Job searching is one of those things where the meta-work — tracking applications, knowing when to follow up, figuring out which companies are ghosts vs slow — takes almost as much energy as the actual applying. I was managing this in a Notion database and spending 20 minutes every few days trying to figure out what needed attention.

This does two things. First, a webhook to log applications as you apply — quick POST, confirmation back, done. Second, a daily 8am email with a pipeline analysis: how many applications, response rate, what needs follow-up today, and draft follow-up emails ready to copy-paste for each one.

The follow-up emails are the part that saves the most time. Claude writes them based on the company, role, how long it's been, and what stage you're at. "Following up after applying" reads differently than "following up after a first-round interview" — both are short, professional, and don't sound like they came from a template.

---

## What it does

**Application logging (webhook):**
- POST an application: company, role, applied date, status, salary range, contact, notes, excitement rating
- Saves to Google Sheets, returns the application ID

**Daily digest (every day at 8am):**
- Loads all applications from Google Sheets
- Identifies which need follow-up (7 days after applying, 5 days if in interview stage, max 2 follow-ups)
- Flags applications that have gone stale (21+ days, still showing as applied)
- Tracks recent outcomes (offers, rejections, ghosts)
- Sends to Claude: pipeline assessment + draft follow-up emails for each application that needs one
- Delivers a formatted digest email with: assessment, today's priority action, pipeline breakdown, ready-to-send email drafts, stale app recommendation
- Only sends if there's actually something that needs attention

---

## Stack

- **n8n** — webhook + daily scheduler
- **Google Sheets** — application database
- **Anthropic Claude** (claude-opus-4-5) — email drafting + pipeline coaching
- **SMTP** — email delivery

---

## Setup

### 1. Create the Google Sheet

One tab: **Applications** — columns:

```
application_id | company | role | applied_date | status | job_url | salary_range | location | remote_policy | contact_name | contact_email | notes | excitement | source | logged_at | follow_up_count | last_follow_up
```

Leave it empty. The webhook fills it.

### 2. Environment variables

```
JOBS_SHEET_ID=your_google_sheet_id
FROM_EMAIL=tracker@yourdomain.com
USER_EMAIL=you@email.com
```

### 3. Credentials

- **Google Sheets OAuth2**
- **Anthropic API**
- **SMTP**

### 4. Import and activate

Import `workflow.json`, activate. Test the webhook first (see below), then wait for 8am tomorrow to see the digest, or trigger the daily check manually.

---

## Logging an application

```bash
curl -X POST https://your-n8n.com/webhook/track-application \
  -H "Content-Type: application/json" \
  -d '{
    "company": "Vercel",
    "role": "Senior Software Engineer",
    "applied_date": "2025-04-29",
    "status": "applied",
    "job_url": "https://vercel.com/careers/...",
    "salary_range": "160k-200k",
    "location": "Remote",
    "remote_policy": "fully remote",
    "contact_name": "Sarah Chen",
    "contact_email": "sarah@vercel.com",
    "notes": "Applied via LinkedIn. Referred by Jamie. Role is on the DX team.",
    "excitement": 8,
    "source": "linkedin"
  }'
```

**Valid statuses:** `applied`, `phone_screen`, `interview`, `final_round`, `offer`, `rejected`, `ghosted`, `withdrawn`

**Required:** `company`, `role`, `applied_date`

---

## Updating application status

The workflow doesn't have a dedicated update endpoint yet. To change a status (e.g. you got a phone screen), find the row in the Google Sheet and update the `status` column directly. The next daily digest will pick up the new status.

A `PATCH /update-application` endpoint is on the roadmap.

---

## Follow-up logic

The workflow follows up:
- After **7 days** if status is still `applied`
- After **5 days** if status is `phone_screen`, `interview`, or `final_round`
- **Maximum 2 follow-ups** per application
- Won't re-follow up within the same window

When you actually send a follow-up, update `follow_up_count` (+1) and `last_follow_up` (today's date) in the sheet. Otherwise the workflow will keep drafting follow-ups for the same application.

---

## The excitement rating

Scale 1–10 when logging an application. This isn't used in the current analysis but it's useful to capture in the moment. When you have 15 active applications and need to decide where to focus energy, sorting by excitement is more useful than sorting by date.

I'll add this to the coaching prompt in a future version.

---

## What the digest looks like

- Header: total apps, active count, response rate
- Pipeline assessment: 3-4 sentences from Claude — honest about what the data shows
- Today's priority: one concrete action
- Status breakdown table: how many at each stage
- Follow-up drafts: one card per application, with subject + body ready to copy
- Stale apps section: what Claude recommends for the zombie applications

---

## The digest only sends when needed

If there are no follow-ups due and nothing stale, the daily check runs but sends nothing. You won't get an empty email every morning just because the workflow is active.

---

## Tracking sources

The `source` field is freeform: `linkedin`, `referral`, `company_website`, `job_board`, `cold_email`, etc. Future versions will break down response rate by source — useful for figuring out whether your LinkedIn applications are actually converting.

---

## Limitations

- Status updates require manual edits in the sheet. There's no `PATCH` endpoint yet.
- The follow-up counter (`follow_up_count`) also requires a manual update when you actually send. If you don't update it, you'll keep getting drafts for the same application.
- Claude doesn't know your specific experience or the company in depth — the follow-up emails are solid templates but you should read them before sending and add a specific detail if you have one.
- No deduplication — if you accidentally POST the same application twice, you'll get two rows. Check before logging.

---

## Ideas

- [ ] `PATCH /update-application` endpoint for status updates
- [ ] Auto-mark as ghosted after 30 days with no movement
- [ ] Weekly summary: response rate by source, average time to first response, interview conversion rate
- [ ] Calendar integration — log scheduled interviews directly to Google Calendar

---

## License

MIT.
