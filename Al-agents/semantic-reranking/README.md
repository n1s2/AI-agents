# FLOOWBOX - Semantic Search Re-Ranking (Brave + Gemini)

Raw search API results rank by keyword match. That's not the same as relevance. This workflow fixes that.

## What it does

Takes an incoming webhook query, runs it through Brave Search API, gets the top results, then sends them to Google Gemini which re-ranks them by semantic relevance to the actual query intent. Returns structured, re-ranked JSON via webhook response.

## Tools Used
- **Orchestration:** n8n
- **Search:** Brave Search API (free tier)
- **Re-ranking:** Google Gemini
- **Trigger / Output:** Webhook

## Flow
```
Webhook Input (query)
  → Brave Search → Raw Results
  → Gemini Semantic Scorer
  → Sorted by true relevance
  → Webhook Response (structured JSON)
```

## Why I built this
Built after noticing that the first result from a search API often wasn't what I actually needed. Adding Gemini's semantic re-ranking step consistently surfaced better results for client research tasks.
