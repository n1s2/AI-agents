# customer-health-scorer

Customer health scores are only useful if they're specific enough to act on. A single number without a breakdown of what's driving it — and what to do about it — just tells you a customer is "at risk" when it's too late to do anything. This scores health across five dimensions, surfaces specific signals, and gives prioritized actions with urgency and owner for each at-risk account. Fires an immediate alert to the CS owner for critical and at-risk accounts.

---

## What it does

Takes customer account data: tier, MRR, renewal date, usage metrics (DAU/WAU/MAU, features adopted, last login, sessions, API calls), support metrics (open tickets, critical tickets, satisfaction), engagement metrics (NPS, CS call frequency, QBR status), financial metrics (expansion, downgrades, payment issues), and CS notes. Claude produces:

- Health score (0–100)
- Health label: healthy / at_risk / critical / churned / champion
- Score components across 5 dimensions: product adoption, CS engagement, support health, financial health, sentiment
- Key strengths and risk signals (specific, not generic)
- Churn probability
- Recommended actions with urgency (this_week/this_month/next_quarter) and owner
- Health narrative (2–3 sentences explaining what drives the score)
- Renewal risk assessment

For critical or at_risk accounts: fires immediate text email alert to CS owner. For all accounts: emails full HTML report with score breakdown bars.

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
curl -X POST https://your-n8n.com/webhook/score-customer-health \
  -H "Content-Type: application/json" \
  -d '{
    "account_id": "ACC-00291",
    "customer_name": "Meridian Logistics",
    "account_tier": "Enterprise",
    "mrr": 2499,
    "currency": "USD",
    "days_as_customer": 412,
    "contract_renewal_date": "2025-09-01",
    "dau": 18,
    "wau": 34,
    "mau": 58,
    "features_used": 6,
    "total_features": 14,
    "last_login_days_ago": 3,
    "sessions_last_30_days": 87,
    "open_tickets": 2,
    "tickets_last_90_days": 7,
    "critical_tickets_last_90_days": 1,
    "avg_response_satisfaction": 3.8,
    "nps_score": 6,
    "last_nps_date": "2025-03-01",
    "cs_calls_last_90_days": 1,
    "last_cs_call_days_ago": 67,
    "qbr_completed": false,
    "expansion_revenue_last_12m": 0,
    "downgrades": 0,
    "payment_issues_last_12m": 0,
    "cs_notes": "Renewal conversation flagged as difficult. Champion Sara Chen left the company in February. New ops lead Jordan hasn't been onboarded yet. Team using basic features only.",
    "cs_owner_email": "alex@company.com",
    "reply_email": "cs-lead@company.com"
  }'
```

**Required:** `customer_name`, `account_id`

---

## Score components

Claude weighs five dimensions:
- **Product adoption** (most predictive): feature breadth, session frequency, DAU/MAU ratio, last login recency
- **CS engagement**: call frequency, QBR completion, responsiveness
- **Support health**: ticket volume, critical ticket count, satisfaction scores
- **Financial health**: expansion vs contraction, payment issues
- **Sentiment**: NPS, CS notes, stated satisfaction

A customer can score high on adoption but low on sentiment — the breakdown shows what's actually driving the overall number.

---

## Limitations

- Analysis is only as good as the data you provide. Pass null/zero for unknown metrics rather than guessing — Claude handles missing data gracefully and notes gaps.
- This is a point-in-time score. Run it periodically (weekly or monthly per account) and compare trends to detect direction of travel.

---

## Ideas

- [ ] Batch scoring: loop through all accounts from a Sheet and score each weekly
- [ ] CRM sync: push health scores back to Salesforce or HubSpot after each scoring run
- [ ] Trend tracking: log scores over time to detect declining trajectories before they become critical
- [ ] Slack alert: post at-risk alerts to a CS Slack channel in addition to email

---

## License

MIT.
