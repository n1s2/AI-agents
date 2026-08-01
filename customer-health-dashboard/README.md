# customer-health-dashboard

A spreadsheet of customer data doesn't tell a CS team who to call this week. This takes up to 50 customers with their health signals — login recency, active users, feature adoption, NPS, support tickets, renewal dates, CSM notes — and produces a prioritized health dashboard: health scores per customer, ARR at risk, priority actions with urgency, expansion opportunities, and specific recommendations for the CS team this week.

---

## What it does

Takes an array of customers with available health signals. Claude analyzes and produces:

- **Portfolio summary** — healthy/at_risk/critical counts, total ARR at risk, renewals in next 90 days, expansion pipeline estimate
- **Health scores** — each customer with: score (0–100), health status (healthy/at_risk/critical), churn risk (low/medium/high), primary risk signal, positive signals, risk signals, recommended action, action urgency, expansion opportunity
- **Priority actions** — ranked list of who to call this week, with specific action, reason, and ARR at stake
- **Portfolio risks** — systemic patterns across customers (e.g., "4 of 6 at-risk customers haven't done a QBR in over 90 days")
- **Expansion opportunities** — customers showing signals for upsell, with estimated expansion ARR
- **CS team recommendations** — specific, numbered actions for the team this week

HTML dashboard with portfolio summary bar, priority action cards, full health table sorted by ARR, and expansion opportunities.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-health-dashboard \
  -H "Content-Type: application/json" \
  -d '{
    "report_name": "June 2025 CS Health Review",
    "report_date": "2025-06-10",
    "currency": "USD",
    "reply_email": "cs@flowdesk.com",
    "health_definitions": "Healthy: login <7d, >70% users active, NPS >7, <2 open tickets. At risk: login 8-21d OR adoption <50% OR NPS 5-6 OR renewal <60d. Critical: login >21d OR NPS <5 OR renewal <30d with risk signals.",
    "customers": [
      {"id": "CUST-001", "name": "Beacon Logistics", "csm": "Priya", "plan": "business", "arr": 14400, "contract_renewal_date": "2025-08-15", "last_login_days": 2, "active_users": 18, "total_users": 22, "feature_adoption_score": 0.82, "support_tickets_open": 0, "support_tickets_last30": 1, "nps_score": 9, "csm_notes": "Strong champion in Tanya. Mentioned adding 10 more users next quarter.", "expansion_opportunity": "team expansion"},
      {"id": "CUST-002", "name": "Meridian Transport", "csm": "Priya", "plan": "business", "arr": 9600, "contract_renewal_date": "2025-07-01", "last_login_days": 18, "active_users": 4, "total_users": 15, "feature_adoption_score": 0.31, "support_tickets_open": 3, "support_tickets_last30": 7, "nps_score": 4, "csm_notes": "Admin user Sarah left 3 weeks ago. No new admin has been set up. Team is confused."},
      {"id": "CUST-003", "name": "Pacific Agency", "csm": "Jake", "plan": "starter", "arr": 3600, "contract_renewal_date": "2025-09-30", "last_login_days": 1, "active_users": 8, "total_users": 10, "feature_adoption_score": 0.65, "support_tickets_open": 1, "nps_score": 7, "csm_notes": "Growing team. Asked about business plan features last call."}
    ]
  }'
```

**Required:** `customers`

---

## Health signals used

Pass whatever signals you have. Claude works with whatever subset is available:

| Signal | Field | What it indicates |
|---|---|---|
| Login recency | `last_login_days` | Engagement drop-off |
| User activation | `active_users` / `total_users` | Adoption rate |
| Feature adoption | `feature_adoption_score` (0–1) | Depth of engagement |
| Support load | `support_tickets_open`, `support_tickets_last30` | Friction or dissatisfaction |
| NPS | `nps_score` | Sentiment |
| Renewal proximity | `contract_renewal_date` | Risk window |
| CSM notes | `csm_notes` | Qualitative context |

More signals = better scoring. The agent works with partial data and flags where signals are missing.

---

## Custom health definitions

Pass your own health thresholds in `health_definitions`. Claude applies your definitions rather than generic defaults. This is useful when your product has different engagement patterns (e.g., a weekly-use tool vs a daily-use tool).

---

## Portfolio risks

Claude identifies patterns across the portfolio that the per-customer view misses — things like "5 of your 8 at-risk customers haven't had a QBR in the last 90 days" or "3 customers flagged lack of executive sponsor in CSM notes." These systemic patterns often require process changes, not just individual customer calls.

---

## Limitations

- Health scoring is based on the signals you provide. Missing signals produce lower-confidence scores — Claude notes when key signals are absent.
- Up to 50 customers per call. For larger portfolios, run by CSM or by segment and synthesize the results.

---

## License

MIT.
