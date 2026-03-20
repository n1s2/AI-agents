# FLOOWBOX - News Site Scraper Without RSS (AI Summary → NocoDB)

Not every important industry site has an RSS feed. This handles those.

## What it does

Scrapes a news website directly using CSS selectors to find article links and content. Filters for posts from the last 7 days. For each article, an LLM generates a summary and extracts keywords. Everything gets saved to a NocoDB table — date, title, summary, keywords, URL.

## Tools Used
- **Orchestration:** n8n
- **Storage:** NocoDB
- **Summarization:** LLM chain
- **Logic:** HTML extraction, date filtering, deduplication

## Why I built this
A client needed daily monitoring of an industry-specific news source that had no RSS feed and no API. This workflow handles it end-to-end on a schedule.

## Customization
Change the CSS selector in the HTML extraction node to match the structure of your target site. The date filter is currently set to 7 days — adjust based on your schedule frequency.
