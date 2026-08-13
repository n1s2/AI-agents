# product-launch-checklist-builder

Launch checklists that live in one team's head fail when that team assumes another team already handled something. This builds a cross-functional launch checklist spanning engineering, product, marketing, sales, support, legal, and design — phased by timeline, with explicit cross-team dependencies, risk items, a launch day runbook, and a rollback trigger defined in advance.

---

## What it does

Takes launch name, date, size (small/medium/major), feature description, target audience, teams involved, and complication flags (pricing change, breaking changes, press outreach). Claude produces:

- **Launch summary** — scope and key complications to manage
- **Phases** — timeline-based (e.g., 4 weeks before, 1 week before, launch day, 1 week after), each with items: specific task, owner team, priority (critical/important/nice_to_have), and details
- **Cross-team dependencies** — what blocks what, between which teams, needed by when
- **Risk items** — with mitigation
- **Launch day runbook** — specific sequential steps for the day itself
- **Rollback trigger** — what would trigger pulling back the launch, decided in advance
- **Success metrics** — with target and measurement window
- **Post-launch review date**

HTML checklist with interactive checkboxes, team-color-coded items, priority badges, and dependency cards.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/build-launch-checklist \
  -H "Content-Type: application/json" \
  -d '{
    "launch_name": "Bulk Task Operations",
    "launch_date": "2025-07-15",
    "launch_size": "medium",
    "feature_description": "Users can select multiple tasks and assign, move, or archive them in one action. New UI in the task list, new bulk-assign API endpoint.",
    "target_audience": "Existing customers with 15+ users, particularly ops-heavy teams currently using workarounds",
    "teams_involved": ["engineering", "product", "marketing", "support", "sales"],
    "has_pricing_change": false,
    "has_breaking_changes": false,
    "press_outreach_planned": false,
    "reply_email": "priya@flowdesk.com"
  }'
```

**Required:** `launch_name`, `launch_date`

---

## Launch size

`small` (minor feature, internal comms only), `medium` (customer-facing feature, coordinated comms), `major` (significant launch, press/marketing push, broader coordination). Size scales the depth and formality of the checklist.

---

## Complication flags shape the checklist

`has_pricing_change` adds items for legal review of pricing terms and customer communication about billing impact. `has_breaking_changes` adds items for migration guides and deprecation timelines. `press_outreach_planned` adds PR-specific items and coordination with the press-release-writer agent's output.

---

## Rollback trigger decided in advance

Like the feature-flag-rollout-planner agent, this asks the team to define what would trigger pulling back the launch before it happens — not as a panic decision made live if something goes wrong on launch day.

---

## Limitations

- The checklist is a comprehensive starting template — always review with each team lead to confirm nothing specific to your organization's process is missing.
- This generates the checklist; tracking actual completion requires your project management tool (import as tasks, or track manually against this document).

---

## License

MIT.
