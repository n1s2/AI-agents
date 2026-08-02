# technical-debt-assessor

"We have a lot of technical debt" doesn't help anyone decide what to fix first. Engineers rank debt by how much it annoys them; product ranks debt as zero priority because it doesn't ship features. This scores each debt item on business impact, velocity cost, and risk — not just code elegance — and gives a business case for each item that engineers can actually use in a conversation with product and leadership.

---

## What it does

Takes system name, context, team size, business context, upcoming roadmap, and up to 40 debt items (each with area, type, effort estimate, incident count, team pain level, and blocked features). Claude produces:

- **Assessment summary** — overall debt picture, biggest risk, biggest opportunity
- **Debt scores** — each item with: business impact score, velocity cost score, risk score, composite priority (P0–P3), business case (argument for the business, not just engineering), cost of inaction, and recommended approach (full_rewrite/incremental_refactor/targeted_fix/monitor_only/accept_as_is)
- **Quick wins** — items with high impact relative to effort
- **Roadmap blockers** — debt items specifically blocking planned features, with urgency
- **Compounding risks** — how specific debt cascades into bigger problems if unaddressed
- **Suggested capacity allocation** — recommended % of engineering time for debt work, with rationale
- **Talking points** — specific arguments for the conversation with product/leadership
- **Debt to accept** — items that are OK to leave as-is for now, with reason

HTML report with score cards showing business impact/velocity cost/risk side by side, quick wins, and prioritized list sorted by tier.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/assess-technical-debt \
  -H "Content-Type: application/json" \
  -d '{
    "system_name": "Task Assignment Service",
    "system_context": "Core service handling task creation, assignment, and status updates. Built in year 1, now serving 3x the original expected load.",
    "team_size": 4,
    "business_context": "This service is on the critical path for our biggest planned feature (bulk operations) and has caused 3 production incidents in the last quarter.",
    "upcoming_roadmap": "Bulk task operations (Q3), real-time collaboration features (Q4)",
    "reply_email": "priya@flowdesk.com",
    "debt_items": [
      {"id": "DEBT-001", "area": "database", "type": "performance", "description": "Task assignment table has no composite index on (project_id, assignee_id), causing full table scans on the most common query pattern as data has grown", "estimated_effort_days": 2, "incident_count": 3, "team_pain_level": "high", "blocking_features": ["bulk task operations"], "notes": "Gets worse every month as table grows"},
      {"id": "DEBT-002", "area": "api", "type": "architecture", "description": "Task status updates are synchronous calls to 4 downstream services (notifications, analytics, search index, audit log). Any one being slow makes the whole request slow.", "estimated_effort_days": 8, "incident_count": 1, "team_pain_level": "medium", "blocking_features": ["real-time collaboration"], "notes": "Would need to move to async event-driven pattern"},
      {"id": "DEBT-003", "area": "code_quality", "type": "code_quality", "description": "TaskService class has grown to 2400 lines handling assignment, notification, validation, and reporting logic all mixed together", "estimated_effort_days": 5, "incident_count": 0, "team_pain_level": "medium", "notes": "Slows down every feature that touches tasks, no incidents but constant friction"},
      {"id": "DEBT-004", "area": "testing", "type": "test_coverage", "description": "No integration tests for the bulk assignment code path — only unit tests with mocked dependencies", "estimated_effort_days": 3, "incident_count": 0, "team_pain_level": "low", "blocking_features": ["bulk task operations"]}
    ]
  }'
```

**Required:** `system_name`, `debt_items`

---

## Business-first scoring

Every score comes with a `business_case` — the argument for why this matters beyond "the code is messy." For DEBT-001 above, the business case connects a database index gap directly to the planned bulk operations feature and three production incidents — something a PM or exec can act on.

---

## Debt to accept

Not all debt should be fixed. Claude explicitly identifies items that are acceptable to leave as-is given current priorities — this is as valuable as the priority list, because it gives engineers permission (and rationale) to not chase every code smell.

---

## Capacity allocation

Claude recommends a specific percentage of engineering capacity to allocate to debt work, with rationale tied to the severity and roadmap risk of the assessed items. This is a starting point for the team's actual capacity planning conversation.

---

## Limitations

- Assessment quality depends on the specificity of debt item descriptions. "The code is messy" produces a generic score. "This causes N+1 queries under load, contributed to 3 incidents, and blocks the bulk operations feature" produces an actionable one.
- This assesses debt as reported — it doesn't scan your actual codebase or detect debt automatically.

---

## License

MIT.
