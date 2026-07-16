# team-standup-summarizer

Async standups generate a wall of individual updates that nobody reads carefully. The team lead skims them, misses the shared blocker that's affecting three people, and doesn't notice the risk to the sprint goal until Friday. This synthesizes individual updates into a team-level view: what's the overall pulse, what's blocking people, what does the manager need to act on today, and a Slack-ready digest to share with the team.

---

## What it does

Takes team name, date, sprint context, and an array of individual updates (yesterday/today/blockers per person). Claude synthesizes:

- **Team pulse** — green/yellow/red with rationale
- **Sprint goal status** — on_track/at_risk/blocked/not_applicable with explanation
- **Collective focus** — themes appearing across multiple people's updates
- **Blockers** — each with who's affected, severity, and suggested action
- **Progress highlights** — notable completions worth calling out
- **Risks** — risks to sprint or team velocity visible across updates
- **Manager follow-ups** — specific things the team lead should do today
- **Async decisions needed** — decisions to make to unblock people
- **Slack digest** — a conversational under-200-word summary ready to post in a channel

HTML output with pulse badge, blocker cards, member update table, and Slack digest block.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/summarize-standup \
  -H "Content-Type: application/json" \
  -d '{
    "team_name": "Platform Engineering",
    "standup_date": "2025-05-31",
    "sprint_name": "Sprint 44",
    "sprint_goal": "Ship Notion integration to production and resolve all P0 support tickets",
    "recipient_emails": ["tom@flowdesk.com", "priya@flowdesk.com"],
    "updates": [
      {"name": "Tom Walsh", "role": "Backend Engineer", "yesterday": "Finished the Notion OAuth flow, deployed to staging", "today": "Writing integration tests for the webhook deduplication logic", "blockers": "Need Amara to review the frontend reconnect flow before I can mark FD-1205 done"},
      {"name": "Amara Nwosu", "role": "Frontend Engineer", "yesterday": "Fixed the Safari iOS logout bug (FD-1189), merged and deployed", "today": "Working on bulk task assignment UI (FD-1195)", "blockers": ""},
      {"name": "Sara Kim", "role": "Backend Engineer", "yesterday": "Investigated the SSO intermittent failure — found the root cause (race condition in token refresh)", "today": "Writing the fix for FD-1207, should have a PR up by noon", "blockers": "FD-1207 fix might need a coordinated deploy with the auth team — need to check with Priya"},
      {"name": "Jeff Lin", "role": "Fullstack Engineer", "yesterday": "Kicked off the analytics schema migration planning", "today": "On PTO today back Monday", "blockers": "Migration is blocked pending DBA review of the new schema — ticket created but no response yet"}
    ]
  }'
```

**Required:** `team_name`, `standup_date`, `updates`

---

## Update format

Each update accepts flexible field names:
- `yesterday` or `done` — what was completed
- `today` or `plan` — what's planned
- `blockers` or `blocked` — what's blocking

At least one of yesterday or today must have content.

---

## Slack digest

The `slack_digest` field is a conversational summary ready to paste into Slack — not a bullet-point list of individual updates, but a synthesized team view. Under 200 words, describes what the team is collectively working on, any blockers, and the day's most important context.

---

## Manager follow-ups

Claude identifies specific things the team lead should act on today — not vague ("check in on progress") but specific ("Sara needs confirmation from Priya about whether FD-1207 fix needs a coordinated deploy before she can finish"). These are derived directly from the updates, not generic advice.

---

## Limitations

- Analysis is only as good as the updates submitted. "Working on stuff" as an update produces a thin synthesis.
- The standup table in the HTML is the raw updates — not edited. The synthesis is the added value.

---

## License

MIT.
