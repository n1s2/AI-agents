# FLOOWBOX - Competitor Price Monitor with AI Analysis

Scrapes competitor pricing pages every day, runs AI analysis comparing their pricing to mine, and sends a daily intelligence report. Always know before your clients ask.

## What it does

Fetches pricing pages from 2 competitors daily. Extracts all text content from both pages. GPT-4o analyzes both, extracts exact plan names and prices, compares them against FLOOWBOX pricing, identifies gaps and opportunities, and recommends whether to adjust pricing. Sends a clean report to my inbox every morning.

## Tools Used
- **Orchestration:** n8n
- **Scraping:** HTTP Request + HTML extraction
- **AI:** OpenAI GPT-4o
- **Email:** SMTP / Gmail
- **Trigger:** Daily schedule

## Flow
```
Daily at set time
  → Fetch Competitor 1 pricing page
  → Fetch Competitor 2 pricing page (parallel)
  → Extract text from both pages
  → Merge both results
  → GPT-4o: extract prices + compare + recommend
  → Email daily report
```

## What the AI report includes

1. Competitor 1 — all plans and exact prices extracted
2. Competitor 2 — all plans and exact prices extracted
3. Comparison vs FLOOWBOX pricing
4. Gaps and opportunities identified
5. Pricing adjustment recommendation

## Why I built this

Clients sometimes ask "why should I pay X when competitor Y charges Z?" — I need to know that answer before they do. This runs automatically and I get the intel in my inbox every morning without thinking about it.

## Setup

1. Add OpenAI API key
2. Add email credentials (Gmail OAuth or SMTP)
3. In "Set Competitor Config" node, replace placeholder URLs with real competitor pricing page URLs
4. Update `my_pricing` with your actual pricing
5. Set `alert_email` to your email

## Extending this

Add a Google Sheets log to track pricing changes over time. Add more competitors by duplicating the fetch + extract nodes and adding them to the merge.
