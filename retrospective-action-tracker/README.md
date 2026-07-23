# retrospective-action-tracker

Retrospectives that produce a list of feelings and vague commitments ("we'll communicate better") don't improve anything. This collects async retro submissions from the whole team during the sprint, then synthesizes them into: specific themes with proposed actions, commitments with owners and verification criteria, recurring issues that appear across multiple sprints, and a team health signal. Two webhooks: one for collecting items throughout the sprint, one for generating the summary.

---

## What it does

**Two-webhook architecture:**

### Webhook 1: `/submit-retro-item`
Team members submit individual retro items asynchronously during the sprint. Each item has a type and gets saved to Google Sheets.

### Webhook 2: `/generate-retro-summary`
Called after the sprint ends (or at retro time). Reads all items for the sprint from Sheets, filters by team/sprint, and Claude synthesizes:

- **Retro summary** — 3–4 sentence sprint narrative
- **Team health** — green/yellow/red with rationale
- **Themes** — clusters of related items, each with category (process/technical/people/communication/tooling), grouped items, a proposed action, owner role, and timeline
- **Sprint wins** — specific things worth calling out
- **Recurring issues** — problems that appear systemic, not one-off
- **Kudos highlights** — recognition worth sharing with the team
- **Commitments table** — action, owner role, due date, how to verify
- **Facilitator notes** — what the team lead should follow up on

---

## Stack

n8n, Google Sheets, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Setup

**Env vars:** `RETRO_SHEET_ID` (Google Sheets ID), `FROM_EMAIL`

**Google Sheet columns:** `item_id`, `team_name`, `sprint_name`, `item_type`, `item_text`, `submitted_by`, `votes`, `timestamp`

---

## Submitting items

```bash
curl -X POST https://your-n8n.com/webhook/submit-retro-item \
  -H "Content-Type: application/json" \
  -d '{
    "team_name": "Platform Engineering",
    "sprint_name": "Sprint 44",
    "item_type": "went_poorly",
    "item_text": "The SSO bug investigation took 3 days because we had no runbook for auth issues. Next time this happens we will be in the same position.",
    "submitted_by": "Sara Kim",
    "votes": 4
  }'
```

**Item types:** `went_well`, `went_poorly`, `action_item`, `kudos`, `blocker`

Items can be submitted any time during the sprint. `votes` defaults to 1 — teams can upvote items before the summary is generated.

---

## Generating the summary

```bash
curl -X POST https://your-n8n.com/webhook/generate-retro-summary \
  -H "Content-Type: application/json" \
  -d '{
    "team_name": "Platform Engineering",
    "sprint_name": "Sprint 44",
    "sprint_goal": "Ship Notion integration to production and resolve all P0 support tickets",
    "velocity_notes": "Committed 42 points, completed 38. SSO investigation consumed 3 unplanned days.",
    "reply_email": "eng-lead@flowdesk.com"
  }'
```

---

## Recurring issues

Claude identifies issues that appear systemic — mentioned by multiple people, high-vote items, or described as patterns by submitters. These are flagged separately from one-off sprint issues so the team can prioritize structural fixes over tactical patches.

---

## Commitments vs themes

Themes group items and propose actions. Commitments are the specific, verifiable things the team agrees to do — each with an owner role (not a name, so it survives personnel changes) and a how-to-verify field so the next retro can check whether it was actually done.

---

## Limitations

- The Google Sheet accumulates items across sprints. Make sure `team_name` and `sprint_name` are consistent when submitting items and generating summaries.
- Synthesis quality improves with more specific items. "Communication was bad" is harder to cluster than "We found out about the API rate limit change the same day it affected production because there was no channel for infra announcements."

---

## License

MIT.
