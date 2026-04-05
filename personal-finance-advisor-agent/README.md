# FLOOWBOX - Personal Finance Advisor Agent

A personal CFO that runs on the first of every month — analyzing your spending, auditing subscriptions, and giving specific actionable advice without you having to open any app.

## What it does

On the 1st of each month, fetches current and previous month transactions from Google Sheets. GPT-4o analyzes spending by category, compares month-over-month changes, calculates savings rate, identifies unusual expenses, audits subscriptions for ones that may be unused, and generates top 3 savings opportunities. Saves the full report to a monthly history sheet and sends an email digest.

## Tools Used
- **Orchestration:** n8n
- **Data:** Google Sheets (transaction log)
- **Analysis:** OpenAI GPT-4o
- **Report:** Email + Google Sheets history
- **Schedule:** Monthly on the 1st

## Flow

```
1st of month, 9 AM
  → Fetch this month + last month transactions
  → Merge both datasets
  → GPT-4o: full financial analysis
  → Save to monthly report sheet
  → Send email digest
```

## Analysis output

```json
{
  "total_income": 85000,
  "total_expenses": 62000,
  "savings": 23000,
  "savings_rate_percent": 27,
  "spending_by_category": [
    {"category": "Food", "amount": 12000, "vs_last_month": "+18%"}
  ],
  "subscription_audit": [
    {"name": "Adobe CC", "amount": 3500, "last_used": "45 days ago", "keep": false}
  ],
  "top_3_savings_opportunities": [
    "Cancel Adobe CC — not used in 45 days (saves ₹3,500/month)",
    "Food spending up 18% — consider meal prep 2 days/week",
    "Consolidate 3 cloud storage subscriptions into one"
  ]
}
```

## Transaction sheet format

| Column | Format |
|---|---|
| Date | YYYY-MM-DD |
| Description | Merchant name |
| Category | Food / Transport / Entertainment / Utilities / Subscriptions / Other |
| Amount | Number |
| Type | income / expense |

## Why I built this

Managing personal finances manually — exporting statements, categorizing, comparing — took 2-3 hours monthly. This does the analysis automatically and surfaces things that are easy to miss, like subscriptions you forgot about.

## Setup

1. Google Sheets with transaction data (two tabs: Current Month + Last Month)
2. OpenAI API key
3. SMTP credentials for monthly email
