# FLOOWBOX - Investment Research Agent

Getting a comprehensive view of any stock or company used to take hours of reading earnings calls, analyst reports, and news. This agent does it in under 2 minutes using three parallel research streams.

## What it does

Send a stock ticker or company name. Three Perplexity AI calls run in parallel — one for recent news and analyst ratings, one for current financial metrics, one for competitive and industry context. GPT-4o synthesizes all three into a structured investment research report with bull case, bear case, key risks, catalysts, analyst consensus, and a plain English summary. Report saves to Notion research library.

## Tools Used
- **Orchestration:** n8n
- **Research (x3):** Perplexity AI Sonar (real-time, parallel)
- **Synthesis:** OpenAI GPT-4o
- **Storage:** Notion
- **Trigger:** Webhook

## Three parallel research streams

| Stream | What it finds |
|---|---|
| Company News | Recent developments, earnings, management changes |
| Financial Data | P/E, revenue growth, margins, EPS, analyst targets |
| Industry Context | Competitive landscape, sector outlook, market position |

## Research report structure

```json
{
  "current_sentiment": "bullish",
  "bull_case": [
    "Market leader in cloud AI infrastructure",
    "Revenue growing 35% YoY",
    "Strong moat from proprietary hardware"
  ],
  "bear_case": [
    "Valuation stretched at 45x forward P/E",
    "Increasing competition from hyperscalers"
  ],
  "analyst_consensus": "buy",
  "avg_price_target": "₹4,200",
  "key_risks": ["Regulatory scrutiny", "Customer concentration"],
  "catalysts": ["Next earnings beat", "New product launch Q3"],
  "investment_summary": "Company shows strong fundamental momentum with..."
}
```

## Why I built this

Investment research was time-consuming because no single source had everything — you'd read news on one site, financials on another, analyst reports somewhere else. Parallel Perplexity calls with GPT-4o synthesis compresses 2 hours into 90 seconds.

## Important disclaimer

This workflow produces AI-generated research for informational purposes only. It is not financial advice. Always conduct your own due diligence and consult a qualified financial advisor before making investment decisions.

## Webhook payload

```json
{
  "ticker": "RELIANCE.NS",
  "depth": "standard",
  "investor_type": "retail"
}
```

## Setup

1. Perplexity API key
2. OpenAI API key
3. Notion integration + Research DB ID
