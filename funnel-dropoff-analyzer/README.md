# FLOOWBOX - Funnel Drop-off Analyzer

Most businesses know their conversion rate but not which specific stage is killing it. This workflow calculates drop-off at every funnel stage, identifies the biggest problem, and gives GPT-4o diagnosed causes with prioritized fixes.

## What it does

Every Monday, pulls funnel stage data from Google Sheets — visits, signups, checkout starts, purchases, etc. Calculates exact drop-off percentage between each stage and flags the biggest single drop-off point. GPT-4o diagnoses likely causes per stage with industry benchmarks, prioritizes which fixes to test first, and estimates the revenue impact of a 10% improvement at the bottleneck stage.

## Tools Used
- **Orchestration:** n8n
- **Data:** Google Sheets (funnel stage visitor counts)
- **Metric Calculation:** Code node
- **Diagnosis:** OpenAI GPT-4o
- **Storage:** Notion
- **Report:** Slack
- **Schedule:** Weekly Monday 9 AM

## Flow

```
Monday 9 AM
  → Fetch funnel stages (Google Sheets)
  → Calculate: visitors per stage, drop-off %, flag biggest drop
  → GPT-4o: diagnose causes + benchmark + fixes
  → Save to Notion
  → Slack report with quick wins
```

## Example diagnosis output

```json
{
  "biggest_problem_stage": "Checkout → Purchase",
  "overall_health": "weak",
  "stage_diagnoses": [
    {
      "stage": "Checkout → Purchase",
      "dropoff_rate": 68,
      "likely_causes": ["Too many form fields", "No trust signals at payment", "Unexpected shipping cost at checkout"],
      "priority_fixes": ["Add SSL badge near payment button", "Show free shipping threshold clearly before checkout", "Reduce form to 5 fields"]
    }
  ],
  "quick_wins": ["Add order summary on checkout page", "Show '30-day money back' guarantee at CTA"],
  "estimated_revenue_impact": "Improving checkout CVR by 10% would add approximately ₹45,000/month at current traffic"
}
```

## Google Sheets format

One row per funnel stage, in order:

| Stage | Visitors |
|---|---|
| Landing Page | 10000 |
| Product Page | 4200 |
| Add to Cart | 1800 |
| Checkout Start | 950 |
| Purchase Complete | 310 |

## Why I built this

A client had a 3% overall conversion rate but didn't know which stage to fix first. After running this workflow, it was clear 67% of people were dropping off at checkout — a shipping cost surprise. Fixing that single step doubled their conversion rate.

## Setup

1. Google Sheets with funnel stage data
2. OpenAI API key
3. Notion integration + DB ID
4. Slack Bot Token + #analytics channel
