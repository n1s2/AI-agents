# FLOOWBOX - Property Inventory Enrichment (AI Vision + Web Agent)

Manual property inventory tagging is slow and inconsistent. This workflow automates the entire enrichment process — photo in, structured data out.

## What it does

Pulls property survey records from Airtable that have attached photos. Sends each photo to OpenAI Vision which generates a structured description — item type, condition, brand indicators, estimated value range. An AI Agent then takes that description and searches the web (via Google Reverse Image Search + Firecrawl scraping) to find matching products, current market prices, and comparable listings. All enriched data gets written back to the Airtable record automatically.

## Tools Used
- **Orchestration:** n8n
- **Vision Model:** OpenAI GPT-4o Vision
- **Web Search:** SERP Google Reverse Image API
- **Page Scraping:** Firecrawl
- **Data Source / Output:** Airtable
- **Logic:** AI Agent with tool calling, fallback response handling

## Flow
```
Airtable (new survey rows with photos)
  → Get applicable rows
  → OpenAI Vision → structured item description
  → AI Agent with tools:
      - Reverse Image Search (SERP API)
      - Web scrape results (Firecrawl)
  → Enrich record with product data + pricing
  → Write back to Airtable
```

## Why I built this

A property management client was paying staff to manually look up every item in their inventory surveys — checking condition, estimating value, finding comparable listings. This workflow replaced that completely. The AI Vision step alone saves 3-4 minutes per item.

## Fallback handling

If the web search returns no useful results, the workflow falls back to the Vision model's own estimate rather than leaving the field blank.

## Commit date
Added: March 2026
