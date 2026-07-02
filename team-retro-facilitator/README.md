# team-retro-facilitator

Retrospectives are most useful when someone synthesizes the raw feedback into themes — five people saying slightly different things about the same problem should become one clear pattern with a proposed fix, not five separate bullet points. This collects retro items asynchronously during or before the meeting, then generates a synthesis: clustered themes, severity-rated problems, concrete action items, and a team health signal.

---

## What it does

**Submit item (webhook `/submit-retro-item`):** Team members submit individual items before or during the retro. Each item is categorized as went_well, went_poorly, action_item, kudos, or blocker. Supports anonymous submission. Saves to Google Sheets.

**Generate summary (webhook `/generate-retro-summary`):** Loads all items for the sprint, groups by category, then Claude clusters similar items into themes (counting frequency), severity-rates problems, proposes concrete action items (specific and assignable — not "improve communication"), highlights kudos, gives a team health signal (strong/stable/concerning/needs_attention), and adds a facilitator note for the scrum master. Emails the team.

---

## Stack

n8n (two webhooks), Google Sheets, Anthropic Claude (claude-sonnet-4-20250514), SMTP.

---

## Setup

**Sheet "Items"** columns: `submitted_at | sprint_name | category | item_text | submitted_by`

**Env vars:** `RETRO_SHEET_ID`, `FROM_EMAIL`

---

## Submitting items

```bash
curl -X POST https://your-n8n.com/webhook/submit-retro-item \
  -H "Content-Type: application/json" \
  -d '{
    "sprint_name": "Sprint 42",
    "category": "went_poorly",
    "item_text": "We kept getting pulled into unplanned support tickets mid-sprint and it broke our focus",
    "submitted_by": "Tom",
    "anonymous": false
  }'
```

**Categories:** `went_well`, `went_poorly`, `action_item`, `kudos`, `blocker`

---

## Generating the summary

```bash
curl -X POST https://your-n8n.com/webhook/generate-retro-summary \
  -H "Content-Type: application/json" \
  -d '{
    "sprint_name": "Sprint 42",
    "team_email": "engineering@company.com",
    "reply_email": "scrummaster@company.com"
  }'
```

Claude clusters all items for that sprint into themes, proposes action items, and emails the formatted summary.

---

## Action item quality

Claude is explicitly instructed to avoid vague actions. "Improve our deployment process" gets rejected in favor of "Add a smoke test step to the CI pipeline that runs before merging to main — owned by platform team, target: Sprint 43." Every action item comes with the theme it addresses and a suggested owner role.

---

## Async-first design

The submit and generate endpoints are decoupled — teams can submit items over days, then generate the summary right before the retro meeting or use it as the meeting artifact. Works equally well for async retrospectives where the team never meets live.

---

## Limitations

- The synthesis is per-sprint, identified by the exact `sprint_name` string. Use consistent naming ("Sprint 42" not "sprint 42" or "S42") or items won't aggregate correctly.
- Action items are proposals — the team should review and assign actual owners during or after the retro. The agent suggests owner roles, not specific individuals.

---

## Ideas

- [ ] Slack collection: add a Slack slash command that posts to the submit webhook so team members don't need to use curl
- [ ] Sprint comparison: generate a trend view across 3–5 sprints showing recurring themes
- [ ] Action item tracker: log proposed action items to a separate Sheet and check completion status at the next retro
- [ ] Jira integration: auto-create tickets for proposed action items with appropriate labels

---

## License

MIT.
