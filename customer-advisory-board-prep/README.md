# customer-advisory-board-prep

Advisory board meetings that just walk through a roadmap slide deck get polite nods, not real feedback. This prepares a meeting structured around genuine discussion — specific open-ended questions per topic (not yes/no validation), honest status updates on previous commitments, and member-specific talking points so the facilitator can draw out perspectives relevant to each person's context.

---

## What it does

Takes company name, meeting date, member list (with title, company, and notes), roadmap items to present, feedback needed, previous meeting action items, and meeting goal. Claude produces:

- **Meeting objective** — what success looks like for this session
- **Suggested agenda** — segments with duration, purpose, and facilitation notes
- **Discussion topics** — each with context to share before asking for feedback, specific open-ended discussion questions, and what good feedback looks like (so the facilitator knows if they're getting real signal or polite validation)
- **Previous action items status** — honest updates to share with the board
- **Member-specific notes** — a talking point tailored to each attendee's use case or background
- **Closing ask** — specific commitments, intros, or follow-up availability to request
- **Facilitator tips** — specific to running a productive discussion with this group

HTML document with agenda timeline, discussion topic cards with questions, and member notes.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/prep-advisory-board \
  -H "Content-Type: application/json" \
  -d '{
    "company_name": "Flowdesk",
    "meeting_date": "2025-07-10",
    "meeting_goal": "Get feedback on the new bulk operations feature before GA, and validate our Q4 roadmap direction",
    "company_context": "Series A, 45 people, focused on ops teams at 15-100 person companies",
    "reply_email": "product@flowdesk.com",
    "members": [
      {"name": "Tanya Okonkwo", "title": "VP Operations", "company": "Beacon Logistics", "notes": "Heavy user of task assignment, has given detailed feedback before, appreciates being asked for specifics"},
      {"name": "David Chen", "title": "Ops Director", "company": "Meridian Consulting", "notes": "More reserved in group settings, tends to give better feedback 1:1 — worth following up separately"},
      {"name": "Priya Patel", "title": "COO", "company": "Pacific Agency", "notes": "New to the board, first meeting, agency-vertical perspective which is underrepresented"}
    ],
    "roadmap_items": [
      "Bulk task operations (launching next month)",
      "Advanced reporting dashboard (Q4)",
      "Mobile app improvements (Q4)"
    ],
    "feedback_needed": [
      "Is the bulk operations UX intuitive or does it need more guidance/onboarding?",
      "How do agencies specifically think about reporting needs vs logistics companies?",
      "Priority ranking of the three Q4 items from their perspective"
    ],
    "previous_action_items": [
      "Ship Slack integration (completed, shipped in May)",
      "Investigate mobile offline mode (still investigating, no committed timeline yet)"
    ]
  }'
```

**Required:** `company_name`, `meeting_date`, `members`

---

## Getting real feedback, not validation

The `what_good_feedback_looks_like` field for each discussion topic helps the facilitator recognize when they're getting genuine signal versus polite agreement — a common failure mode in advisory board meetings where members don't want to seem unhelpful.

---

## Member-specific talking points

Each member gets a note tailored to their context — Tanya's history of detailed feedback, David's tendency toward reserved group behavior (suggesting a follow-up 1:1), Priya's underrepresented agency perspective. This helps the facilitator actively draw out quieter members and follow up appropriately after the meeting.

---

## Honest previous action item updates

If a committed item (like the mobile offline mode investigation) hasn't progressed, Claude generates language for reporting that honestly rather than glossing over it — advisory boards notice when their input disappears into a black hole, and honest status updates build trust even when the news isn't "done."

---

## Limitations

- Prep quality depends on how much context you provide about each member and their history with the product. Richer notes produce more useful member-specific talking points.
- This prepares the meeting; running it well still depends on the facilitator's skill in the room.

---

## License

MIT.
