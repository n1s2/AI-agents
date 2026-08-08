# customer-usage-report-generator

Raw usage numbers ("450 API calls, 12 active users, 3.2 GB storage") mean nothing to a customer or a CSM without translation into business value. This turns usage data into a narrative report — connecting metrics to outcomes, honestly assessing feature utilization, checking capacity against plan limits, and adjusting tone based on whether the audience is the customer themselves or internal CS/leadership.

---

## What it does

Takes customer name, usage data (any JSON structure), plan limits, prior period data for comparison, account value, renewal date, and audience type (customer-facing or internal). Claude produces:

- **Report title** and **executive summary**
- **Key metrics** — each with value, trend (up/down/flat/new), and context explaining what it means
- **Utilization analysis** — well-utilized vs underutilized features, overall verdict (strong/moderate/weak)
- **Business value narrative** — 2–3 paragraphs connecting usage to business outcomes, tone-adjusted for audience
- **Capacity and limits** — current usage vs plan limits with headroom assessment
- **Comparison to prior period** — honest trend narrative if data provided
- **Recommendations** — with rationale, flagged as audience-appropriate
- **Renewal readiness note** — for internal audience only, honest risk assessment based on usage patterns

HTML report with metric cards showing trend arrows, well/underutilized feature comparison, and capacity table.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-usage-report \
  -H "Content-Type: application/json" \
  -d '{
    "customer_name": "Beacon Logistics",
    "product_name": "Flowdesk",
    "report_period": "Q2 2025",
    "audience_type": "internal",
    "account_value": 14400,
    "currency": "USD",
    "contract_renewal_date": "2025-08-15",
    "csm_name": "Priya Sharma",
    "reply_email": "priya@flowdesk.com",
    "usage_data": {
      "active_users": 18,
      "total_licensed_users": 22,
      "tasks_created": 1240,
      "tasks_completed": 1180,
      "projects_active": 12,
      "integrations_connected": ["Slack"],
      "avg_session_duration_minutes": 14,
      "logins_per_week_avg": 3.2,
      "api_calls_last_30d": 0
    },
    "plan_limits": {
      "users": 25,
      "projects": "unlimited",
      "storage_gb": 100
    },
    "prior_period_data": {
      "active_users": 20,
      "tasks_created": 1450,
      "logins_per_week_avg": 4.1
    }
  }'
```

**Required:** `customer_name`, `usage_data`

---

## Audience-appropriate tone

`audience_type: "customer"` produces a partnership-oriented narrative suitable to send directly to the customer — highlighting value delivered, framing gaps as opportunities. `audience_type: "internal"` produces a more direct assessment including the `renewal_readiness_note` field, which honestly flags risk signals (declining logins, low integration adoption near a renewal date) that wouldn't be appropriate to share with the customer directly.

---

## Usage data is flexible

Pass any JSON structure for `usage_data` — the agent works with whatever metrics your product tracks. No fixed schema required.

---

## Limits and headroom

If you pass `plan_limits`, Claude assesses current usage against them and gives a headroom assessment — useful for spotting expansion signals (customer near their user limit) or right-sizing conversations (customer using a fraction of a higher tier).

---

## Limitations

- The narrative quality depends on how much usage data you provide. Sparse data produces a thin report.
- This generates the report content — it doesn't pull data from your product analytics platform directly. Feed it data from your existing analytics/BI pipeline.

---

## License

MIT.
