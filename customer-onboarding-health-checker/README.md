# customer-onboarding-health-checker

Most churn from failed onboarding is preventable — the warning signs show up weeks before the customer actually leaves, but nobody's watching milestone completion against expected timeline until it's too late. This tracks onboarding milestones against expected timing, distinguishes benign delays from actual stalls, identifies specific blockers, and tells the CSM exactly what to do next and how urgently.

---

## What it does

Takes customer name, signup date, onboarding milestones (each with expected day, completed day or status, and critical path flag), team size, active users, CSM assigned, and recent contact/notes. Claude produces:

- **Onboarding health** — on_track/at_risk/stalled/completed with summary
- **Completion percent**
- **Milestone analysis** — each milestone's status (completed_on_time/completed_late/overdue/upcoming/not_started), days variance from expected, blocking flag, and notes
- **Blockers identified** — specific blocker, likely cause, and suggested action
- **Churn risk from onboarding** — high/medium/low/none with rationale
- **Recommended next action** — specific action, urgency (immediate/this_week/monitor), and who should act (csm/support/product/customer)
- **Projected completion** — realistic estimate given current pace

HTML report with health status badge, progress bar, milestone table with variance column, and next action card color-coded by urgency.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/check-onboarding-health \
  -H "Content-Type: application/json" \
  -d '{
    "customer_name": "Meridian Consulting",
    "signup_date": "2025-05-15",
    "days_since_signup": 31,
    "expected_onboarding_days": 21,
    "team_size": 15,
    "active_users": 3,
    "csm_assigned": "Priya Sharma",
    "account_value": 7200,
    "currency": "USD",
    "last_customer_contact": "Kickoff call on day 3, no contact since. Two follow-up emails sent, no response.",
    "customer_notes": "Champion (ops director) went on leave shortly after signup. Not clear who the new point of contact is.",
    "reply_email": "priya@flowdesk.com",
    "onboarding_milestones": [
      {"name": "Account setup", "expected_day": 1, "completed_day": 1, "critical_path": true},
      {"name": "First project created", "expected_day": 3, "completed_day": 4, "critical_path": true},
      {"name": "Team members invited", "expected_day": 5, "completed_day": null, "status": "not_started", "critical_path": true},
      {"name": "First task completed", "expected_day": 7, "completed_day": null, "status": "not_started", "critical_path": true},
      {"name": "Integration configured", "expected_day": 14, "completed_day": null, "status": "not_started", "critical_path": false},
      {"name": "First weekly review", "expected_day": 21, "completed_day": null, "status": "not_started", "critical_path": true}
    ]
  }'
```

**Required:** `customer_name`, `onboarding_milestones`

---

## Milestone format

Each milestone needs `expected_day` (days from signup) and either `completed_day` (if done) or `status`. Mark `critical_path: true` for milestones that block progress if not completed — the analysis treats these differently from optional milestones.

---

## Blockers vs benign delays

Claude distinguishes between a milestone that's a few days late (often benign — team is just busy) and a genuine blocker (champion left, technical issue, confusion about next steps). The example above shows exactly this pattern: team members not invited plus no customer contact for weeks plus the champion going on leave adds up to a real blocker, not just slow pace.

---

## Churn risk from onboarding specifically

This isn't general customer health scoring — it's specifically about whether onboarding trajectory itself predicts churn. A customer who's 10 days behind but actively engaged with support is different from a customer who's gone silent. The `churn_risk_from_onboarding` field and rationale reflect this distinction.

---

## Limitations

- Assessment quality depends on how complete your milestone tracking is. Partial milestone data produces a partial picture — the more milestones tracked, the more accurate the health assessment.
- This assesses a single customer at a time. For portfolio-wide onboarding health, run per customer and aggregate, or see the customer-health-dashboard agent for portfolio views.

---

## License

MIT.
