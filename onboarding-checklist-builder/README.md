# onboarding-checklist-builder

Bad onboarding is either too much on day one (new hire is overwhelmed before they can find the bathroom) or a ghost experience (here's your laptop, figure it out). This generates a phased onboarding checklist with separate manager tasks and new hire tasks side by side, realistic timing, tool provisioning priorities, key people to meet, and what good looks like at 30 and 90 days.

---

## What it does

Takes role title, company name, department, onboarding type, tool stack, team context, and role responsibilities. Claude generates:

- **Onboarding summary** — what this plan achieves and what success looks like
- **30-day and 90-day success definitions** — measurable outcomes, not vibes
- **Phases** — each with timeframe, theme, and two parallel task lists:
  - Manager tasks (with timing and notes)
  - New hire tasks (with timing and notes)
  - End-of-phase milestones
- **Tools to set up** — each with access type, who provisions it, and priority (day 1/week 1/month 1)
- **Key people to meet** — role, why this meeting matters, suggested timing
- **Common onboarding mistakes** — specific to this role type, not generic advice

HTML output shows manager and new hire tasks side by side with interactive checkboxes, color-coded phase headers, and tool priority badges.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/build-onboarding-checklist \
  -H "Content-Type: application/json" \
  -d '{
    "role_title": "Senior Backend Engineer",
    "company_name": "Flowdesk",
    "department": "Engineering",
    "onboarding_type": "new_hire",
    "start_date": "2025-07-07",
    "manager_name": "Priya Sharma",
    "buddy_name": "Tom Walsh",
    "remote_or_onsite": "fully_remote",
    "company_stage": "Series A, 45 people",
    "checklist_duration_days": 60,
    "tool_stack": ["Slack", "GitHub", "Linear", "Notion", "Flowdesk", "Datadog", "AWS", "Zoom"],
    "team_context": "4-person backend team. Very async. We use Linear for task tracking and Notion for docs. Architecture decisions made in Slack threads. Code review is the main sync point.",
    "role_responsibilities": "Own the integrations layer. First project: Notion sync feature currently in staging. Expected to be shipping independently by week 4.",
    "key_milestones": "First PR merged by day 5. First feature shipped independently by day 30. Owning integrations roadmap by day 60.",
    "reply_email": "priya@flowdesk.com"
  }'
```

**Required:** `role_title`, `company_name`

---

## Onboarding types

`new_hire`, `internal_transfer`, `contractor`, `rehire`, `executive`

Type shapes the checklist. Executives get a stakeholder-mapping focus in week one. Contractors get a streamlined version without culture-building activities. Internal transfers skip the "here is what we do" basics and focus on the new team and responsibilities.

---

## Manager vs new hire tasks

The two-column layout makes it clear who owns what. Managers often forget that some of their tasks (access provisioning, intro emails, schedule setup) have to happen before day one or the new hire sits idle. The checklist calls this out explicitly with timing notes.

---

## Tool provisioning

The `tools_to_set_up` section assigns priority levels: day_1 (must have before they start), week_1 (need in first week to do their job), month_1 (useful but not urgent). Flagged with who provisions — prevents the situation where the new hire doesn't have GitHub access because the manager assumed IT handled it.

---

## Limitations

- The checklist is a starting point — every role has company-specific context that should be added. The template handles structure; the manager adds specifics.
- For highly regulated industries (finance, healthcare, legal), compliance onboarding steps should be added separately with input from legal/compliance teams.

---

## License

MIT.
