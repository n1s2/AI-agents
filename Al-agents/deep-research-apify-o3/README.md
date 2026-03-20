# FLOOWBOX - Self-Hosted Deep Research Agent (Apify + OpenAI o3)

The upgraded version of my research agent — built when Jina AI wasn't cutting it for JS-rendered pages.

## What it does

Full autonomous research pipeline. User inputs a query, the agent generates multiple targeted searches, uses Apify actors to scrape and render pages that block normal HTTP requests, then runs OpenAI o3 for multi-step reasoning before generating a final structured report.

## Tools Used
- **Orchestration:** n8n
- **LLM:** OpenAI o3
- **Scraping:** Apify actors
- **Logic:** Agentic reasoning loop

## Why I built this
Some high-value pages (dashboards, gated content, SPAs) just don't work with simple HTTP fetches. Apify handles the browser rendering layer — o3 handles the reasoning. No compromises on data quality.

## When to use this vs the standard Deep Research workflow
Use this one when your target pages are JavaScript-heavy or require browser rendering. Use the standard one (SerpAPI + Jina) for most regular research tasks — it's faster and cheaper.
