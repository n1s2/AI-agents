# FLOOWBOX - Google Analytics AI Weekly Report → Baserow

Agency clients don't want GA dashboards — they want one paragraph saying what changed and what to do about it. This generates that automatically.

## What it does

Pulls Google Analytics 4 data for the current week and previous week — page engagement stats and Google Search Console results. Parses and structures both datasets, sends them to an LLM for side-by-side comparison analysis, then saves the AI-written report to Baserow. Runs every Monday morning.

## Tools Used
- **Orchestration:** n8n
- **Data:** Google Analytics 4 API + Search Console
- **Analysis:** LLM via OpenRouter
- **Storage:** Baserow

## Why I built this
Manual reporting was the biggest time sink in client work. This eliminates it completely — the LLM output is good enough to send directly to clients after a quick review.

## Setup
You need a Google Analytics credential in n8n (OAuth). Set your GA4 Property ID in the node config. Baserow table needs Date, Report, and Client columns.
