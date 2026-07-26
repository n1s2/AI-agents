# meeting-notes-extractor

Meeting notes written by hand are either too long (full transcript) or too short (a list of bullet points that only make sense to people who were there). This takes a raw transcript and extracts structured notes: a summary useful to people who weren't in the meeting, decisions with rationale, action items with owners and deadlines, open questions, risks raised, and a polished email-ready summary ready to send to attendees or stakeholders.

---

## What it does

Takes meeting transcript, title, type, date, and attendees. Claude extracts:

- **Summary** — 3–5 sentences useful to someone who wasn't in the meeting
- **Key topics** — tag-style overview of what was covered
- **Decisions** — each with rationale (why it was decided, not just what), who decided it, and when it takes effect
- **Action items** — task, owner, due date, context (why this action is needed), priority (high/medium/low)
- **Open questions** — who needs to answer each, and by when
- **Risks raised** — concerns mentioned during the meeting
- **Follow-up meetings** — suggested follow-ups with purpose, attendees, and timing
- **Email summary** — polished 150-word meeting summary with subject line, ready to send

Can auto-send to a list of recipients (attendees, stakeholders, etc.) immediately after processing.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/extract-meeting-notes \
  -H "Content-Type: application/json" \
  -d '{
    "meeting_title": "Sprint 44 Planning",
    "meeting_type": "team_meeting",
    "meeting_date": "2025-06-05",
    "attendees": ["Priya Sharma", "Tom Walsh", "Amara Nwosu", "Sara Kim", "Jeff Lin"],
    "context": "Sprint planning for a 14-day sprint. Team is 4 engineers. Previous sprint delivered 38 of 42 committed points.",
    "output_formats": ["summary", "action_items", "decisions"],
    "recipient_emails": ["priya@flowdesk.com", "eng-team@flowdesk.com"],
    "reply_email": "priya@flowdesk.com",
    "meeting_transcript": "[Paste the full meeting transcript here]"
  }'
```

**Required:** `meeting_transcript`, `meeting_title`

---

## Meeting types

`team_meeting`, `customer_call`, `executive_review`, `design_review`, `sales_call`, `1on1`, `all_hands`, `incident_review`, `strategy`, `interview`, `other`

Type shapes what Claude focuses on. Customer calls emphasize commitments made and next steps. Design reviews emphasize decisions and open questions. Incident reviews emphasize root causes and action items. 1-on-1s emphasize action items and follow-ups.

---

## Decisions with rationale

Claude captures not just what was decided but why — the rationale behind the decision. "We decided to defer the analytics migration" is less useful than "We decided to defer the analytics migration because the DBA review hasn't completed and we don't want to block the sprint on external dependency." Six months later, when someone asks why, the rationale is there.

---

## Email summary

The `email_summary` field is a polished, standalone 150-word summary with a subject line on the first line. It's written for people who may not have been in the meeting — clear, complete, no jargon. Paste it directly into an email or Slack message.

---

## Connecting to meeting tools

For fully automated note-taking, connect this to Zoom, Google Meet, or Fireflies webhooks — when a meeting recording is processed, the transcript gets sent here automatically and notes are emailed to attendees before they've closed their laptop.

---

## Limitations

- Transcript is capped at 10,000 characters (~1,500 words). For longer meetings, pass the most substantive section or a pre-summarized version.
- Action item ownership is extracted from names mentioned in the transcript. If the transcript uses pronouns without clear referents, ownership may be assigned to "TBD."

---

## License

MIT.
