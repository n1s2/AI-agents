# FLOOWBOX - Umami Analytics Week-over-Week AI Report → Baserow

Privacy-first analytics with AI-generated insights. Umami gives me the data — this workflow tells me what it means.

## What it does

Pulls Umami website stats for the current week (pageviews, visitors, bounces, total time) and the previous week. Separately fetches per-page traffic data for both periods. Sends summary stats to LLM for an overview analysis. Sends page-level data to LLM for a week-over-week comparison with 5 improvement suggestions. Both reports saved to Baserow. Runs every Thursday.

## Tools Used
- **Orchestration:** n8n
- **Analytics:** Umami API (self-hosted)
- **AI:** Llama 3.1 70B via OpenRouter (free tier)
- **Storage:** Baserow

## Why I built this
Umami is my preferred analytics tool — privacy-respecting, self-hosted, no Google. This workflow adds the AI interpretation layer on top of the raw numbers.
