# FLOOWBOX - Open Deep Research Agent

My self-hosted alternative to paid deep research tools. Takes a question, does real multi-step research, writes a full report.

## What it does

Chat input triggers the pipeline. An LLM generates 4 distinct search queries, SerpAPI runs all of them in batches, Jina AI fetches full page content for each result, another LLM extracts relevant context per page, all context is merged, and a final LLM writes a structured Markdown research report with sources.

## Tools Used
- **Orchestration:** n8n
- **LLM:** Google Gemini 2.0 Flash (via OpenRouter)
- **Search:** SerpAPI (Google)
- **Content Fetch:** Jina AI reader API
- **Memory:** n8n Buffer Window (session-based)
- **Knowledge:** Wikipedia tool

## Flow
```
Chat Input
  → Generate 4 targeted queries (LLM)
  → Batch SerpAPI searches
  → Fetch full pages (Jina AI)
  → Extract relevant context per page (LLM)
  → Merge all context
  → Generate final structured report (LLM)
```

## Why I built this
Wanted full research capability on my own infra. OpenRouter lets me swap models without touching the workflow — currently on Gemini 2.0 Flash, can switch to anything.

## API Keys Needed
- OpenRouter
- SerpAPI
- Jina AI
