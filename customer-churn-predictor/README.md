# FLOOWBOX - Customer Churn Predictor

By the time a customer says they're leaving, it's usually too late. This workflow identifies at-risk customers weeks before they churn — when there's still time to do something about it.

## What it does

Every Monday, pulls all active customers from Airtable and cross-references with product usage data from Google Sheets. GPT-4o scores each customer on a 0-100 churn risk scale based on login frequency trends, feature usage depth, support ticket volume, contract age, and payment history. High-risk customers trigger an immediate Slack alert with specific recommended actions. A full report goes to email.

## Tools Used
- **Orchestration:** n8n
- **CRM Data:** Airtable
- **Usage Data:** Google Sheets
- **AI Scoring:** OpenAI GPT-4o
- **Alerts:** Slack (#churn-alerts)
- **Report:** Email
- **Schedule:** Weekly Monday 9 AM

## Flow

```
Monday 9 AM
  → Fetch active customers (Airtable)
  → Fetch weekly usage data (Google Sheets)
  → Merge both datasets
  → GPT-4o scores each customer (0-100)
  → Parse: high / medium / low risk
  → Slack alert for high risk (immediate action)
  → Weekly email report (all customers)
```

## Churn score signals GPT-4o uses

- Login frequency declining week-over-week
- Feature usage dropping below baseline
- Support tickets increasing
- No activity in 14+ days
- Late payments in last 2 months
- Contract approaching renewal with no engagement

## Output per customer

```json
{
  "customer_name": "Acme Corp",
  "churn_score": 78,
  "risk_level": "high",
  "top_signals": ["No login in 12 days", "Support tickets up 3x"],
  "recommended_action": "Schedule executive check-in call this week",
  "urgency": "immediate"
}
```

## Why I built this

A SaaS client lost 4 customers in one quarter who had all shown the same usage drop pattern 6 weeks earlier. Nobody noticed because there was no system to look. This workflow would have caught all 4 in time to intervene.

## Setup

1. Airtable: Customers table with Status, Contract Date, Payment History fields
2. Google Sheets: Weekly usage log (logins, feature events, support tickets)
3. OpenAI API key
4. Slack Bot Token + #churn-alerts channel
5. SMTP credentials
