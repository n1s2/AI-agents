# FLOOWBOX - SEO Rankings AI Weekly Brief (SERPBear → Baserow)

SERPBear tracks rankings perfectly. It just doesn't tell you what to *do* about them. This adds that layer.

## What it does

Fetches keyword ranking data from my self-hosted SERPBear instance via API. Calculates 7-day average position and trend direction (improving/declining/stable) per keyword. Sends the structured data to Llama 3.1 70B (free tier via OpenRouter) for analysis. Saves the weekly SEO brief to Baserow every Monday.

## Tools Used
- **Orchestration:** n8n
- **SEO Data:** SERPBear API (self-hosted)
- **Analysis:** Llama 3.1 70B via OpenRouter (free tier)
- **Storage:** Baserow

## Why I built this
I wanted weekly SEO insights without opening any dashboard. The LLM analysis consistently surfaces things I would have missed looking at raw position numbers.

## Cost
Near zero — Llama 3.1 70B free tier via OpenRouter handles the analysis. SERPBear is self-hosted. Baserow free tier is enough for this data volume.
