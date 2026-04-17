# FLOOWBOX - Revenue Forecast Agent

Monthly revenue forecasting is one of the most important — and most often skipped — business activities for small companies. This workflow does it automatically with proper statistical trend analysis and three-scenario planning.

## What it does

On the 1st of every month, fetches 12 months of revenue history and the current sales pipeline. Runs a linear regression to calculate growth slope and trend direction. Generates weighted pipeline value based on deal probability. GPT-4o creates a three-scenario forecast — base case, optimistic, and pessimistic — with the assumptions behind each, risk factors, growth levers, and recommended focus areas.

## Tools Used
- **Orchestration:** n8n
- **Data:** Google Sheets (revenue history + pipeline)
- **Statistical Analysis:** Code node (linear regression)
- **Forecast Generation:** OpenAI GPT-4o
- **Storage:** Google Sheets (forecast log)
- **Report:** Slack
- **Schedule:** Monthly on 1st

## What the code node calculates

```
Linear regression on last 12 months:
  slope = (n·Σxy - Σx·Σy) / (n·Σx² - (Σx)²)
  intercept = (Σy - slope·Σx) / n
  
3-month projection = slope × (n + i) + intercept

Weighted pipeline = Σ(deal_value × probability%)
```

## Three-scenario output

```json
{
  "base_case": [
    {"month": "May 2026", "forecast": 285000, "confidence": "high"},
    {"month": "Jun 2026", "forecast": 298000, "confidence": "medium"}
  ],
  "optimistic_case": [
    {"month": "May 2026", "forecast": 340000, "assumption": "2 enterprise deals in pipeline close"}
  ],
  "pessimistic_case": [
    {"month": "May 2026", "forecast": 230000, "risk": "Current client renewals delayed"}
  ],
  "trend_direction": "accelerating",
  "growth_levers": ["Activate dormant enterprise leads from Q4", "Launch referral program for existing clients"]
}
```

## Why I built this

FLOOWBOX was running without a formal revenue forecast — decisions about hiring, marketing spend, and sales effort were based on gut feel. This workflow gave me a data-backed projection every month and helped identify that the growth rate was actually decelerating 2 months before it showed up as a problem.

## Setup

1. Google Sheets: Monthly Revenue (Month, Revenue) + Sales Pipeline (Deal, Deal Value, Probability %) sheets
2. OpenAI API key
3. Slack Bot Token + #finance channel
