# meeting-agenda-generator

Meetings without agendas waste time. Meetings with agendas that are just topic lists ("Q3 planning", "Budget discussion") also waste time — nobody knows what decision needs to be made or what "done" looks like for each item. This generates agendas with a desired outcome per item, realistic time allocation, and a clear meeting-level goal — so the meeting actually produces something.

---

## What it does

Takes meeting title, purpose, attendees, duration, topics to cover, decisions needed, and prior meeting context. Claude builds:
- Meeting goal (the single most important outcome)
- Agenda items with time allocation, desired outcome per item (not just topic), owner, and type (decision/discussion/update/brainstorm/action_review)
- Pre-read suggestions
- Parking lot guidance for off-topic items
- Success criteria (how to know the meeting worked)
- Facilitator notes specific to the meeting type

Time allocations sum to approximately the requested duration. HTML output with color-coded item types and prominent time blocks.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-agenda \
  -H "Content-Type: application/json" \
  -d '{
    "meeting_title": "Q3 Roadmap Planning",
    "meeting_purpose": "Decide on the top 3 priorities for Q3 engineering roadmap and get alignment from product and design leads",
    "meeting_type": "decision_making",
    "attendees": ["Priya Sharma (PM)", "Tom Walsh (Eng)", "Amara Nwosu (Design)", "Jeff Lin (CEO)"],
    "duration_minutes": 60,
    "topics_to_cover": ["Review Q2 roadmap completion rate", "Present 3 candidate Q3 priorities", "Discuss resourcing constraints", "Vote and finalize top 3"],
    "decisions_needed": ["Final list of Q3 priorities", "Resourcing allocation across priorities"],
    "organizer_name": "Priya Sharma",
    "reply_email": "priya@company.com"
  }'
```

**Required:** `meeting_title`, `meeting_purpose`, `attendees`, `duration_minutes`

---

## Meeting types

`status_update`, `decision_making`, `brainstorm`, `planning`, `retrospective`, `one_on_one`, `client_meeting`, `board_meeting`, `kickoff`, `all_hands` — each calibrates agenda structure and item types.

---

## License

MIT.
