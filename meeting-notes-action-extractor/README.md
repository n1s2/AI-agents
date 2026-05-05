# meeting-notes-action-extractor

Every team I've worked on has had the same problem: someone takes notes during a meeting, those notes get pasted into a doc nobody reads, and three days later people are asking "wait, who was supposed to do that thing?"

This takes raw meeting notes — however messy — and extracts everything that matters: action items with owners and priorities, decisions made, open questions, blockers, and a parking lot for things that got kicked down the road. It builds a clean summary email, sends it to whoever attended, posts a condensed version to Slack, and logs everything to Google Sheets.

The extraction is the part I spent the most time on. Claude is specifically told to be conservative — if something is ambiguous, it flags it as a question rather than guessing. If a name is mentioned with a task, it captures the attribution. It distinguishes between things that were decided and things still being discussed.

---

## What it does

1. Accepts a POST: meeting title, type, notes, attendees, project name, emails to send to, Slack channel
2. Claude extracts:
   - 3-4 sentence summary
   - Action items: task, owner, due date, priority (urgent/high/medium/low), context
   - Key decisions with rationale and impact
   - Open questions with who asked and when answer is needed
   - Blockers with what's affected and who needs to resolve
   - Parking lot items (deliberately deferred topics)
   - Whether a follow-up meeting is needed + suggested agenda
3. Builds a formatted HTML summary email with a color-coded action items table
4. If attendee emails provided: sends the summary email
5. If Slack channel provided: posts a condensed version with action items and blockers
6. Logs to Google Sheets
7. Returns full extracted JSON in the webhook response

---

## Stack

- **n8n** — webhook + workflow
- **Anthropic Claude** (claude-opus-4-5) — extraction and summarization
- **SMTP** — email delivery
- **Slack** — channel post (optional)
- **Google Sheets** — action item log (optional)

---

## Setup

### 1. Environment variables

```
FROM_EMAIL=meetings@yourcompany.com
MEETINGS_SHEET_ID=your_sheet_id   # optional, for logging
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP**
- **Slack API** (optional)
- **Google Sheets OAuth2** (optional)

### 3. Google Sheet (optional)

Create a sheet called **Action Items** with columns:
```
meeting_date | meeting_title | action_items_json | total_actions | urgent_actions | blockers_count | logged_at
```

### 4. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/process-meeting \
  -H "Content-Type: application/json" \
  -d '{
    "meeting_title": "Sprint Planning — Week 19",
    "meeting_type": "planning",
    "meeting_date": "2025-05-05",
    "project_name": "Payments Overhaul",
    "attendees": ["Priya", "Tom", "Dani", "Rashid"],
    "meeting_notes": "Kicked off at 10am. Tom walked through the velocity from last sprint — 34 points, slightly under the 38 target. Agreed this was fine given the two sick days.\n\nPriya raised that the payment gateway integration is still blocked on credentials from the finance team. Rashid said he would chase that today.\n\nScope for this sprint: complete the refund flow (Dani taking lead), write unit tests for the auth module (Tom), and start the reconciliation report (Priya). All three should be done by end of Friday.\n\nDani flagged that the design for the refund confirmation email still isn\'t final. Needs sign-off from marketing. Priya will ping Emma in marketing today.\n\nWe talked briefly about whether to add the retry logic this sprint. Decided to defer — too risky to scope it without the gateway credentials working first. Will revisit in the next planning.\n\nNext planning is set for Monday 12th.",
    "send_summary_to": ["priya@company.com", "tom@company.com", "dani@company.com", "rashid@company.com"],
    "slack_channel": "#payments-team"
  }'
```

**Required:** `meeting_title`, `meeting_notes`

---

## Meeting types

`standup`, `planning`, `retrospective`, `client_call`, `one_on_one`, `all_hands`, `brainstorm`, `interview`, `other`

The type is passed to Claude for context — it helps calibrate what to look for. A retrospective surfaces different things than a client call.

---

## Note quality matters

Claude extracts what's in the notes. Vague notes produce vague output. Specifically:

- If someone's name is mentioned when assigning a task, Claude captures the attribution. If tasks aren't attributed, they show up as "unassigned".
- If a deadline is mentioned ("by Friday", "end of next week"), it gets captured. If no deadline is mentioned, it shows as "not specified".
- If the reason for a decision isn't in the notes, Claude doesn't invent one.

You don't need perfect notes. "Rashid will chase the credentials today" is enough. But "someone will handle the credentials" produces "unassigned."

---

## Priority levels

Claude assigns priority based on language and context in the notes:
- **Urgent**: things described as blocking, needed immediately, or explicitly called out as critical
- **High**: important tasks with near-term deadlines
- **Medium**: standard work items
- **Low**: nice-to-haves, future work, no deadline pressure

You can override priorities manually by editing the extracted JSON before it routes — but in practice the classification is solid for well-written notes.

---

## The Slack post

The Slack version is a condensed plain-text summary: the meeting summary, the action item list (ID, task, owner, due date, and a 🔴 for urgent items), and the blockers list. It truncates at Slack's message limits but covers all action items.

It's intentionally different from the email — the email is for reference, the Slack post is for "what do I need to do right now."

---

## Parking lot vs open questions

These are distinct:
- **Open questions**: things raised that still need an answer before work can proceed
- **Parking lot**: topics that came up but were deliberately set aside and don't need immediate resolution

Claude keeps these separate rather than collapsing them both into "open items."

---

## Integrating with Notion or Linear

The webhook response includes the full extracted JSON. If you want to push action items to Notion, Linear, Jira, or any other PM tool, add nodes after the **Log to Sheets** step that read from `$('Parse Extraction').first().json.action_items` and create tasks in your tool of choice.

The data structure is designed for this — each action item has id, task, owner, due_date, priority, and context.

---

## Limitations

- Works on text notes. Audio transcription isn't built in — if you have a recording, run it through Whisper or another transcription service first, then paste the transcript.
- Claude reads up to 8,000 characters of notes. Very long meetings may need to be split.
- Owner attribution only works if names appear in the notes. If you have a habit of not naming people in notes, action items will come back as "unassigned."
- Decision confidence varies — Claude is conservative and will mark something as a decision only if it's clearly stated. Implied decisions may show up as open questions instead.

---

## Ideas

- [ ] Recurring meeting mode: compare this meeting's action items against last week's, flag anything that was assigned but not mentioned as complete
- [ ] Whisper integration: accept audio file URL and transcribe before extracting
- [ ] Linear/Jira push: create actual tasks from action items automatically
- [ ] Personal view: filter the email to show only the action items assigned to the recipient

---

## License

MIT.
