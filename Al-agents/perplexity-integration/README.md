# FLOOWBOX - Perplexity AI Integration for n8n

Sometimes you just need real-time web-grounded answers inside a larger workflow. This gives you that in one clean node.

## What it does

Passes a system prompt + user prompt to Perplexity's Sonar model with optional domain filtering. Returns the grounded response + citations array, cleaned and ready to use in the next node.

## Tools Used
- **Orchestration:** n8n
- **AI:** Perplexity AI (Sonar model)

## Setup
Add a Header Auth credential in n8n:
- Name: `Authorization`
- Value: `Bearer YOUR_PERPLEXITY_KEY`

## Why I built this
Rather than spinning up a full search + scrape pipeline for every real-time query, this single API call handles it. Perplexity's grounding is solid for current events and recent data.

## Use cases I plug this into
- Client Q&A bots that need current pricing/news
- Competitive research sub-flows
- Any workflow that benefits from a "what's happening right now" layer
