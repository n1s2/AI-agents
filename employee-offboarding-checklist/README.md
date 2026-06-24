# employee-offboarding-checklist

Employee departures create real operational risk if handled ad-hoc. Forgotten system access is a security issue. Missing knowledge transfer leaves teams unable to do their jobs. Skipped legal steps create compliance exposure. And a poor exit experience damages employer brand and referral networks.

This generates a complete, phase-organized offboarding checklist calibrated to the departure type (voluntary, involuntary, layoff, retirement, contract end), days remaining, role, systems, and whether the employee is client-facing. Sends to manager, HR, or both.

---

## What it does

1. Accepts a POST: employee name, role, department, last day, departure type, manager, HR contacts, systems access, direct reports, client-facing flag, equipment to return
2. Claude generates a complete offboarding package:
   - Summary of key considerations for this specific offboarding
   - Phase-organized checklist: Immediate → First 48 hours → Week 1 → Final week → Last day
   - Each task with owner (HR/Manager/IT/Employee/Legal/Finance), urgency flag, and notes
   - Full access revocation list (using systems provided + role-inferred standard ones)
   - Knowledge transfer topics specific to the role
   - Client transition steps (if client-facing)
   - Legal and compliance items (NDA reminders, benefits continuation, IP agreements)
   - Equipment return list
   - Exit interview questions calibrated to role and departure type
   - Manager talking points for the offboarding conversation
3. Emails checklist to manager email, HR email, and/or reply email
4. Returns full JSON

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP.

---

## Setup

**Env vars:** `FROM_EMAIL`

**Credentials:** Anthropic API (LangChain node), SMTP.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/offboard-employee \
  -H "Content-Type: application/json" \
  -d '{
    "employee_name": "Jordan Park",
    "role": "Senior Software Engineer",
    "department": "Platform Engineering",
    "last_day": "2025-06-13",
    "departure_type": "voluntary",
    "manager_name": "Priya Sharma",
    "manager_email": "priya@company.com",
    "hr_email": "hr@company.com",
    "systems_access": ["GitHub", "AWS", "Datadog", "PagerDuty", "Slack", "Notion", "Linear", "Google Workspace"],
    "direct_reports": 0,
    "client_facing": false,
    "company_equipment": ["MacBook Pro 14\"", "Monitor", "YubiKey", "Corporate credit card"],
    "reply_email": "priya@company.com"
  }'
```

**Required:** `employee_name`, `last_day`, `role`, `department`

---

## Departure types and their effect

| Type | Key differences |
|---|---|
| `voluntary` | Standard timeline, full knowledge transfer window, positive exit framing |
| `involuntary` | Access revocation moves to immediate phase, shorter knowledge transfer window |
| `layoff` | Similar to involuntary but includes severance documentation, may have legal hold periods |
| `retirement` | Extended transition, successor mentoring, longer handoff window |
| `contract_end` | Focus on deliverables completion, IP transfer, final invoice |

For `involuntary` and `layoff`, access revocation is flagged as URGENT in phase 1 — it's the first item rather than a later step.

---

## Access revocation list

Claude uses the `systems_access` array you provide and supplements with systems standard for the role. A Senior Software Engineer gets GitHub and AWS even if you forgot to list them. The list is formatted for an IT ticket — complete, specific, and includes role-inferred items.

---

## Knowledge transfer topics

These are role-specific, not generic. A Senior Software Engineer's knowledge transfer list covers codebase areas owned, runbooks, on-call rotation handoff, vendor relationships with infrastructure providers. A Sales Account Executive's list covers active pipeline, client relationships, territory context. Claude infers from the role.

---

## Exit interview questions

5–6 questions calibrated to the departure type and role. Voluntary departures get questions about decision factors and what could have changed it. Involuntary get questions about process feedback. All include a question about team dynamics and one about what the company should keep doing.

---

## Manager talking points

What to cover in the offboarding conversation — not a script but the key topics the manager should address: what the company appreciates, transition expectations, reference policy, keeping the relationship positive after departure.

---

## Limitations

- The access revocation list is comprehensive but not exhaustive — verify against your actual system inventory before closing the IT ticket. Specialized tools not mentioned and not inferable from the role won't appear.
- For involuntary terminations involving performance issues or legal disputes, involve your employment attorney before following any generated checklist. Offboarding steps can have legal implications.
- The checklist is generated once. It doesn't track completion or send reminders as tasks come due.

---

## Ideas

- [ ] HR system integration: push the checklist as tasks into BambooHR, Rippling, or Workday
- [ ] IT ticket creation: auto-create an access revocation ticket in Jira/ServiceNow with the access list
- [ ] Completion tracking: a companion webhook that marks individual tasks complete and notifies the manager
- [ ] HRIS trigger: trigger the offboarding generation automatically when a termination record is created

---

## License

MIT.
