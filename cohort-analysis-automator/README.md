# FLOOWBOX - Cohort Analysis Automator

Cohort analysis is how you understand whether your product is actually retaining customers or just churning through them. This workflow builds the full retention table from raw order data — no BI tool required.

## What it does

On the 5th of every month, fetches all order history from Google Sheets. The code node groups customers by their first purchase month (cohort) and tracks which subsequent months each customer remained active. Builds a retention matrix showing what percentage of each cohort returned at period 1, period 2, period 3, and beyond. GPT-4o analyzes the patterns — identifying the "churn cliff" period where most customers drop off, whether retention is improving or declining across recent cohorts, and specific interventions for the weakest periods.

## Tools Used
- **Orchestration:** n8n
- **Data:** Google Sheets (order history)
- **Cohort Building:** Code node (custom retention matrix)
- **Analysis:** OpenAI GPT-4o
- **Storage:** Notion
- **Report:** Slack
- **Schedule:** Monthly on 5th

## What the cohort table looks like

| Cohort | Size | Period 0 | Period 1 | Period 2 | Period 3 |
|---|---|---|---|---|---|
| Jan 2026 | 45 | 100% | 42% | 31% | 24% |
| Feb 2026 | 38 | 100% | 47% | 35% | — |
| Mar 2026 | 52 | 100% | 39% | — | — |

## Analysis output

```json
{
  "churn_cliff_period": "Period 1 (first 30 days after signup)",
  "retention_trend": "improving",
  "avg_customer_lifetime_months": 4.2,
  "cohort_insights": [
    "Feb cohort has 12% better Period 1 retention — coincides with improved onboarding email sent that month",
    "Customers who purchase twice in Period 0 have 3x Period 3 retention"
  ],
  "intervention_recommendations": [
    {"period": "Period 1", "action": "Send personalized value-realization email at day 25", "expected_impact": "5-8% retention improvement"}
  ]
}
```

## Why I built this

FLOOWBOX was acquiring customers at a healthy rate but couldn't tell whether retention was improving. The cohort analysis revealed a specific "churn cliff" at Period 1 — most customers who churned did so before their second month. Fixing the onboarding sequence improved Period 1 retention by 18%.

## Setup

1. Google Sheets: Orders sheet with Customer Email, Order Date, Order Value
2. OpenAI API key
3. Notion integration + Analytics DB
4. Slack Bot Token + #analytics channel
