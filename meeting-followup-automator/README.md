# meeting-followup-automator

The follow-up is where most meetings actually die. People leave with a shared understanding that evaporates by the next morning, action items get written in personal notebooks and never tracked, and the decisions that were made get relitigated at the next meeting because nobody sent a clear summary.

This takes your raw notes — whatever you typed during the meeting — and turns them into a structured follow-up: decisions made, action items with owners and due dates, open questions, blockers, and a complete follow-up email ready to send. It also logs key topics, suggests next meeting agenda items, and sends the email automatically if you want.

Two minutes to paste your notes. Zero time to write the follow-up.

---

## What it does

1. Accepts a POST: meeting title, raw notes, attendees, meeting type, organizer, follow-up recipients
2. Claude processes the notes and extracts:
   - 2–3 sentence executive summary
   - Decisions made (with context and decision-maker)
   - Action items (task, owner, due date, priority, notes)
   - Open questions (with who can answer them)
   - Key topics discussed
   - Blockers identified
   - Follow-up email: subject line + complete body, ready to send
   - Next meeting agenda suggestions
3. Builds a structured HTML summary report
4. If `send_follow_up_email` is true: sends the follow-up email to `follow_up_recipients`
5. If `reply_email` is provided: emails the full structured report
6. Returns full JSON with all extracted data

---

## Stack

- **n8n** — webhook + workflow
- **Anthropic Claude** (claude-sonnet-4-20250514) — meeting processing
- **SMTP** — follow-up email + report delivery

---

## Setup

### 1. Environment variables

```
FROM_EMAIL=meetings@yourcompany.com
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP**

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/meeting-followup \
  -H "Content-Type: application/json" \
  -d '{
    "meeting_title": "Q3 Product Roadmap Review",
    "meeting_type": "internal_team",
    "meeting_date": "2025-05-12",
    "attendees": ["Priya Sharma (PM)", "Tom Walsh (Engineering)", "Amara Nwosu (Design)", "Jeff Lin (CEO)"],
    "organizer": "Priya Sharma",
    "organizer_email": "priya@company.com",
    "raw_notes": "Jeff opened by saying the mobile app launch needs to move from Q4 to Q3 end of October. This came from board pressure after competitor launched last week. Tom flagged this is aggressive — we need to cut scope. Agreed to drop offline sync feature for v1 and push it to Q1. Amara will have updated designs for the simplified flow by May 20. Tom to provide revised eng estimate by May 19 EOD. Jeff wants to see the updated plan by May 22 before he goes to board. Open question: do we need to bring in a contractor for the auth module? Tom thinks maybe — he will assess this week. Nobody brought up marketing — Priya to check in with Sam about launch plan timeline.",
    "company": "Acme Corp",
    "project_name": "Mobile App",
    "next_meeting_date": "2025-05-22",
    "send_follow_up_email": true,
    "follow_up_recipients": ["priya@company.com", "tom@company.com", "amara@company.com", "jeff@company.com"],
    "tone": "professional",
    "reply_email": "priya@company.com"
  }'
```

**Required:** `meeting_title`, `raw_notes`, `attendees`

---

## Meeting types

`client_meeting`, `internal_team`, `sales_call`, `project_kickoff`, `board_meeting`, `one_on_one`, `discovery_call`, `retrospective`, `planning`, `interview`, `other`

The type calibrates tone and structure. A board meeting summary is more formal and executive-focused than a team retro.

---

## The follow-up email

The `follow_up_email` in the output is a complete, ready-to-send email — not a template. It:
- Has a specific subject line ("Action items from Q3 Product Roadmap Review — May 12" not "Meeting follow-up")
- Leads with the most important decisions or outcomes
- Lists action items with owners and due dates in bullet format
- Notes any open questions
- Signs off with the organizer's name
- Is under 300 words — people actually read it

If `send_follow_up_email` is true and `follow_up_recipients` is provided, it goes out automatically. If false, the email draft is returned in the JSON for you to review and send manually.

---

## Action item extraction

Claude assigns owners based on the names in the `attendees` list. It looks for explicit ownership ("Tom will…", "Amara to…", "assigned to Jeff") and implicit ownership (whoever raised the topic, whoever has the relevant role). If ownership is genuinely ambiguous, the owner is set to "TBD."

Priority (high/medium/low) is inferred from language and context — "before Jeff goes to the board" gets high, "check in with Sam when you have a chance" gets low.

---

## Open questions

Not everything gets resolved in a meeting. Claude extracts questions that were explicitly left open, and identifies who has the information to answer them. These appear separately from action items — they're not tasks yet, but they need to become tasks.

---

## Tone options

`professional` — standard business register, suitable for any context
`formal` — conservative tone, good for legal, finance, or senior leadership meetings
`casual` — friendlier register, good for team meetings where formality feels stiff
`executive` — tight and direct, minimal context-setting, for C-suite readers

---

## Two emails: follow-up vs report

The workflow can send two different things:
1. **Follow-up email** (`send_follow_up_email: true`): the plain-text action-item email that goes to all meeting participants. This is what everyone opens.
2. **Full report** (`reply_email`): the structured HTML summary with all sections. This goes to whoever submitted the notes — typically the organizer who wants the full picture.

---

## Notes format

Paste your notes in whatever format you have them. Claude handles:
- Stream-of-consciousness bullet points
- Partial sentences
- Names mentioned casually ("Jeff said...")
- Mixed topics
- Notes taken in Notion, Google Docs, or a notebook and transcribed

The only hard limit is 6,000 characters. For very long meetings, summarize or split.

---

## Limitations

- Claude infers owners and priorities from the notes. If your notes are vague ("someone will look into this"), the action item will have an ambiguous owner. More specific notes produce more useful action items.
- Date extraction from relative references ("by next Friday", "end of sprint") requires knowing the meeting date — make sure `meeting_date` is set correctly.
- The workflow doesn't integrate with task management tools (Jira, Asana, Linear) out of the box. To auto-create tasks, add nodes after **Parse Output** that call the relevant API with the action items.

---

## Ideas

- [ ] Jira/Linear integration: auto-create tickets from action items with the extracted owner mapped to a project member
- [ ] Google Calendar follow-up: if next_meeting_date is provided, create a calendar event with the suggested agenda
- [ ] Slack digest: post the decisions and action items to a Slack channel instead of (or alongside) the email
- [ ] Meeting history sheet: log all processed meetings to Google Sheets for tracking and search

---

## License

MIT.
