# noise-complaint-logger

Built this for a friend who manages a 40-unit apartment building and was keeping noise complaints in a WhatsApp group. Residents would message her, she'd screenshot it, sometimes forget to follow up, and then when someone asked "this is the third time I've complained about 4B" she had no record of the first two.

This is a webhook that takes a complaint submission, saves it to a Google Sheet, checks whether the same location has been complained about before, determines an escalation level based on severity and recurrence, and uses Claude to draft the appropriate communications: an acknowledgement to the reporter, and a formal notice to the building manager when the situation warrants it.

The escalation logic is the useful part. A single mild complaint just gets logged and acknowledged. Three complaints from the same unit in 30 days, or anything above severity 7, automatically generates a formal draft and alerts the manager.

---

## What it does

1. Accepts a POST with complaint details: location, noise type, severity (1–10), description, whether it's ongoing
2. Validates and saves to Google Sheets
3. Loads complaint history and checks for repeat incidents from the same location (last 30 days)
4. Determines escalation level:
   - **None**: isolated, low-severity complaint
   - **Warning letter**: 2+ complaints or severity ≥ 5
   - **Building manager**: 3+ complaints or severity ≥ 7
   - **Immediate**: severity ≥ 9 or ongoing + severity ≥ 7
5. Sends to Claude which writes: internal summary, risk assessment, reporter acknowledgement email, formal notice draft (if needed), recommended next steps
6. If reporter email provided: sends acknowledgement
7. If escalation needed: emails the manager with the draft formal notice and full context
8. Returns a JSON confirmation with reference number and escalation status

---

## Stack

- **n8n** — webhook + workflow
- **Google Sheets** — complaint log
- **Anthropic Claude** (claude-opus-4-5) — communication drafting
- **SMTP** — email

---

## Setup

### 1. Create the Complaints sheet

One tab called **Complaints** with these columns:

```
logged_at | reporter_name | reporter_email | location | unit | noise_type | severity | description | start_time | ongoing_now | previous_complaints | status
```

### 2. Environment variables

```
COMPLAINTS_SHEET_ID=your_sheet_id
FROM_EMAIL=complaints@yourbuilding.com
MANAGER_EMAIL=manager@yourbuilding.com
```

### 3. Credentials

- **Google Sheets OAuth2**
- **Anthropic API**
- **SMTP**

### 4. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/noise-complaint \
  -H "Content-Type: application/json" \
  -d '{
    "reporter_name": "James Okafor",
    "reporter_email": "james@email.com",
    "location": "12 Maple Street",
    "unit": "4A",
    "noise_type": "music",
    "severity": 7,
    "description": "Bass music playing since 11pm, clearly from the unit above (4B). Can feel it through the floor. Has happened three times this week.",
    "ongoing_now": true,
    "start_time": "2025-04-28T23:00:00",
    "previous_complaints": 2
  }'
```

**Noise types:** `music`, `construction`, `party`, `dog_barking`, `vehicle`, `shouting`, `machinery`, `other`

**Severity scale:** 1 = barely noticeable, 10 = genuinely unbearable / affects safety

---

## Building a submission form

The webhook is just a POST endpoint. A basic HTML form with Fetch works fine. For a building, you'd probably embed it in your resident portal or send residents a link to a Typeform/Tally form that posts to this webhook.

Tally.so works particularly well — it's free, looks professional, and has a native webhook integration. Map the form fields to the webhook payload fields.

---

## Escalation thresholds

Adjust these in the **Analyse Pattern** node:

```js
if (current.severity >= 9 || (current.ongoingNow && current.severity >= 7)) {
  escalationLevel = 'immediate';
} else if (repeatCount >= 3 || current.severity >= 7) {
  escalationLevel = 'building_manager';
} else if (repeatCount >= 2 || current.severity >= 5) {
  escalationLevel = 'warning_letter';
}
```

Change thresholds to match your building's policies and tolerance levels.

---

## What the formal notice looks like

Claude writes a professional, legally neutral notice. It references building rules in general terms (it doesn't know your specific lease terms — you'd want to review and add your actual lease clause numbers before sending). It avoids naming the reporter. It states the facts: type of noise, time, frequency, and what's expected going forward.

The notice goes to the manager's email as a draft inside the escalation alert. The manager decides whether to print it, email it, or slip it under a door. The workflow never sends it directly to the offending party — that decision stays with a human.

---

## The `previous_complaints` field

This is an optional self-reported number from the submitter ("I've complained about this before"). The workflow also counts complaints independently from the sheet. Both feed into Claude's context. If a resident says "this is the 5th time" but the sheet only shows 2 prior complaints, Claude sees both numbers and can factor that into the tone of the acknowledgement.

---

## Multi-building setup

If you manage several properties, add a `building_id` field to the webhook and sheet. Filter the history lookup in **Analyse Pattern** by building_id. Route escalation emails to different managers based on building. All straightforward to extend.

---

## Limitations

- Pattern matching on location is fuzzy string matching (first 15 characters). If your address data is inconsistent ("12 Maple St" vs "12 Maple Street") you'll get split histories. Standardize location names in your submission form.
- The formal notice is a draft. It should always be reviewed by a human before delivery. Claude doesn't know your local tenancy laws.
- This doesn't track resolution. Add a `resolved_at` column and a separate webhook (`/resolve-complaint`) to close out cases if you want a full lifecycle.

---

## Ideas

- [ ] Weekly complaint summary report for the building committee
- [ ] Map view if you collect GPS coordinates
- [ ] Resident portal integration — residents can check status of their own complaints
- [ ] Auto-close complaints after 7 days with no follow-up

---

## License

MIT.
