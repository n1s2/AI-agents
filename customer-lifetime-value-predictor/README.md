# FLOOWBOX - Customer Lifetime Value Predictor

Not all customers are equal. This workflow calculates RFM scores, predicts 12-month CLV for every customer, and segments them so you know exactly who to prioritize, who to re-engage, and who is about to churn.

## What it does

On the 1st of every month, fetches all order history from Google Sheets. Calculates RFM metrics per customer — Recency (days since last purchase), Frequency (number of orders), and Monetary (average order value). GPT-4o analyzes the patterns to predict 12-month CLV and assign each customer to a segment: Champions, Loyal, At-Risk, Lost, or New. Identifies specific churn signals in the At-Risk segment and recommends actions per segment. Monthly report saves to Sheets and posts a Slack digest.

## Tools Used
- **Orchestration:** n8n
- **Data:** Google Sheets (order history)
- **RFM Calculation:** Code node
- **CLV Prediction:** OpenAI GPT-4o
- **Storage:** Google Sheets (CLV report)
- **Report:** Slack
- **Schedule:** Monthly on 1st

## RFM metrics calculated

| Metric | What it measures |
|---|---|
| Recency | Days since last purchase (lower = better) |
| Frequency | Total number of orders |
| Monetary | Average order value per transaction |

## Customer segments

| Segment | Profile |
|---|---|
| Champions | Recent buyers, buy often, spend most |
| Loyal | Regular buyers, decent spend |
| At-Risk | Used to buy often, haven't recently |
| Lost | No recent purchases, fading |
| New | 1-2 purchases, recent |

## Output

```json
{
  "segment_summary": {"champions": 12, "loyal": 34, "at_risk": 18, "lost": 25, "new": 11},
  "total_predicted_revenue_12m": 285000,
  "at_risk_signals": ["No purchase in 45+ days despite previous 2-week cadence", "Last 2 orders were smaller than baseline"],
  "segment_actions": [
    {"segment": "at_risk", "recommended_action": "Send win-back email with 15% loyalty discount"},
    {"segment": "champions", "recommended_action": "VIP early access to new products"}
  ]
}
```

## Orders sheet schema

| Column | Format |
|---|---|
| Customer Email | Email address |
| Customer Name | Full name |
| Order Date | YYYY-MM-DD |
| Order Value | Number |

## Setup

1. Google Sheets with order history
2. OpenAI API key
3. Slack Bot Token + #ecommerce channel
