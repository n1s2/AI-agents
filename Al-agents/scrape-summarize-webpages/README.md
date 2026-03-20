# FLOOWBOX - Bulk Webpage Scraper and AI Summarizer

Clean, lightweight workflow for processing large numbers of URLs into readable summaries. I use a version of this for content monitoring.

## What it does

Fetches HTML from a list of URLs, extracts page titles, runs each through a GPT-4o-mini summarization chain using a recursive text splitter for long content, then merges title + summary + URL into clean output objects.

## Tools Used
- **Orchestration:** n8n
- **LLM:** OpenAI GPT-4o-mini
- **Logic:** HTTP fetch, HTML extraction, summarization chain, recursive text splitter

## Flow
```
Trigger
  → Fetch URL list
  → Split into individual URLs
  → Limit (configurable for testing)
  → Fetch each page HTML
  → Extract title (parallel)
  → Run summarization chain (parallel)
  → Merge title + summary
  → Clean output (title, summary, URL)
```

## Why I built this
Competitor blog monitoring — need summaries of 20+ articles per week without reading everything. GPT-4o-mini is fast and cheap enough for this use case.
