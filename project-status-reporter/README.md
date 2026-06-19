# project-status-reporter

Weekly status reports are one of those things that take 45 minutes to write and 2 minutes to read. Most of that 45 minutes is organizing raw notes from multiple workstreams into something coherent and stakeholder-readable.

This takes structured updates from each area of your project — what's done, what's planned, blockers, risks, budget — and produces a formatted status report with executive summary, per-area status indicators, narrative synthesis, completed/planned split, and a budget progress bar. Emails directly to your stakeholder list.

Works for any project type: software development, construction, marketing campaigns, product launches, organizational initiatives.

---

## What it does

1. Accepts a POST: project name, reporting period, report type, overall status, status updates by area, completed items, planned items, blockers, risks, budget, recipients
2. Claude synthesizes all updates into:
   - Executive summary (3–4 sentences, 30-second read)
   - Status narrative (2–3 paragraphs synthesizing the area updates into a coherent picture)
   - Period highlight (single most important accomplishment)
   - Key risk (most important concern, or null if all is well)
   - Blocker summary
   - Next period focus
   - Action items needed from stakeholders
3. Builds HTML report with:
   - Overall status badge (ON TRACK / AT RISK / DELAYED / BLOCKED / COMPLETED / ON HOLD)
   - Per-area status cards with color-coded indicators
   - Completed/planned items split
   - Blockers and risks columns
   - Budget progress bar (if budget data provided)
4. Emails to all recipients
5. Returns full JSON

---

## Stack

- **n8n** — webhook + workflow
- **Anthropic Claude** (claude-sonnet-4-20250514) — report writing
- **SMTP** — email delivery

---

## Setup

### 1. Environment variables

```
FROM_EMAIL=reports@yourcompany.com
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP**

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/project-status \
  -H "Content-Type: application/json" \
  -d '{
    "project_name": "Platform API Modernization",
    "project_description": "Migrating legacy REST APIs to GraphQL with improved authentication and rate limiting",
    "reporting_period": "Week of May 12-16, 2025",
    "report_type": "weekly",
    "overall_status": "at_risk",
    "project_manager": "Priya Sharma",
    "status_updates": [
      {
        "area": "Backend Development",
        "status": "on_track",
        "update": "GraphQL schema for user and billing endpoints is complete. Auth middleware integration is 80% done — target completion Friday.",
        "metrics": "14 of 18 endpoints migrated",
        "owner": "Tom Walsh"
      },
      {
        "area": "Frontend Integration",
        "status": "at_risk",
        "update": "Apollo Client integration is running 3 days behind due to unexpected TypeScript compatibility issues discovered Monday. Team is working through them but we may need to push the frontend integration milestone.",
        "owner": "Amara Nwosu"
      },
      {
        "area": "Testing & QA",
        "status": "on_track",
        "update": "Test coverage at 87% for migrated endpoints. Performance testing shows p99 latency improved 40% vs legacy. No critical bugs open.",
        "metrics": "87% test coverage, p99 latency -40%",
        "owner": "Marcus Lee"
      }
    ],
    "completed_this_period": [
      "GraphQL schema finalized for all user endpoints",
      "Rate limiting implementation complete and tested",
      "Legacy API deprecation notices sent to 3 internal consumers"
    ],
    "planned_next_period": [
      "Complete auth middleware integration",
      "Resolve TypeScript compatibility issues in frontend",
      "Begin migration of payment endpoints"
    ],
    "blockers": [
      "Frontend TypeScript compatibility issue — Amara investigating, needs 1-2 days"
    ],
    "risks": [
      "Frontend integration milestone (May 23) may slip by 3-5 days if TS issue takes longer than expected"
    ],
    "budget": 180000,
    "budget_spent": 94000,
    "currency": "USD",
    "recipients": ["engineering-leads@company.com", "product@company.com", "cto@company.com"],
    "reply_email": "priya@company.com"
  }'
```

**Required:** `project_name`, `reporting_period`, `status_updates`

---

## Overall status options

| Status | When to use |
|---|---|
| `on_track` | Everything proceeding as planned |
| `at_risk` | A concern exists that could impact timeline or scope |
| `delayed` | Timeline has slipped or will slip |
| `blocked` | Work cannot proceed without external resolution |
| `completed` | Project or phase is done |
| `on_hold` | Work paused intentionally |

The overall status drives the color of the header status badge. Each area in `status_updates` can also have its own status using the same values.

---

## Status updates structure

Each item in `status_updates` represents one workstream or functional area:

```json
{
  "area": "Backend Development",
  "status": "on_track",
  "update": "What happened and where things stand — be specific",
  "metrics": "Optional quantitative measure",
  "owner": "Person or team responsible"
}
```

You can have 2–10 areas. Claude synthesizes across all of them in the narrative section.

---

## Report types

`weekly`, `bi_weekly`, `monthly`, `milestone`, `executive`, `stakeholder`

The type appears in the report header and email subject. For executive or stakeholder reports, Claude tends to write more concisely and less technically in the narrative.

---

## Action items for stakeholders

Claude identifies things needed from the people receiving the report — decisions, approvals, resources. These appear in a highlighted section at the bottom. If nothing is needed, the section doesn't appear.

This is the most actionable part of a status report and the most commonly omitted.

---

## Budget tracking

Pass `budget` (total) and `budget_spent` so far and a budget progress bar appears in the report. The bar turns orange above 75% and red above 90% of budget used. Currency defaults to USD.

---

## Automating from a form or sheet

To automate weekly status collection:
1. Set up a Google Form or Tally form for each area owner to fill in
2. Aggregate responses in Google Sheets
3. Use an n8n Google Sheets trigger that fires when the sheet is updated
4. Reshape the sheet data to match the webhook body format
5. Call this webhook to generate and send the report automatically

This removes the PM bottleneck — area owners fill in their own status, the report generates and sends itself.

---

## Limitations

- Claude synthesizes from what you provide. Vague updates ("made progress on X") produce vague narratives. Specific updates ("migrated 14 of 18 endpoints, auth middleware 80% complete") produce specific reports.
- The report is generated once per submission. There's no versioning or change tracking between reports.
- No project management tool integration out of the box. To pull status from Jira, Linear, or Asana, add API call nodes before the Claude step to fetch current sprint data.

---

## Ideas

- [ ] Google Sheets trigger: auto-generate on form submission from area owners
- [ ] Jira integration: pull sprint velocity and open issues into the status update
- [ ] Status history: log each report to Google Sheets with overall status and key metrics for trend tracking
- [ ] Slack summary: post a condensed version to a Slack channel alongside the email

---

## License

MIT.
