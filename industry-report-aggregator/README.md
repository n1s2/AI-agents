# FLOOWBOX - Industry Report Aggregator

Staying current on your industry used to mean subscribing to expensive newsletters or spending hours reading reports. This workflow aggregates market intelligence, funding activity, and regulatory changes every Monday — automatically.

## What it does

Every Monday morning, three Perplexity AI searches run in parallel — one finding the week's market reports and statistics from sources like Gartner, McKinsey, and CB Insights; one tracking VC funding rounds and acquisitions; one monitoring regulatory developments. GPT-4o compiles all three into a structured weekly brief including FLOOWBOX-specific opportunities. Saves to Notion and posts a digest to Slack.

## Tools Used
- **Orchestration:** n8n
- **Intelligence (x3):** Perplexity AI Sonar (parallel, weekly filter)
- **Synthesis:** OpenAI GPT-4o
- **Storage:** Notion
- **Digest:** Slack
- **Schedule:** Weekly Monday 7 AM

## Three intelligence streams

| Stream | Coverage |
|---|---|
| Market Reports | Analyst reports, market size, growth projections, statistics |
| Funding Activity | VC rounds, Series A/B/C, acquisitions, M&A |
| Regulatory Updates | Policy changes, compliance requirements, government initiatives |

## Weekly brief output

```json
{
  "market_highlights": [
    "Global AI automation market projected to reach $25B by 2027 (Gartner)"
  ],
  "funding_rounds": [
    {"company": "Make.com", "amount": "$50M", "stage": "Series C", "significance": "Direct competitor expanding"}
  ],
  "regulatory_changes": ["EU AI Act enforcement begins Q2 2025"],
  "emerging_trends": ["Agentic AI adoption accelerating in SMB segment"],
  "opportunities_for_floowbox": [
    "EU AI Act compliance tooling — underserved market",
    "SMB segment expanding faster than enterprise"
  ]
}
```

## Why I built this

Running FLOOWBOX requires staying current on the market. Missing a competitor funding round or regulatory change can mean being caught off-guard in client conversations. This delivers a complete Monday morning briefing — I start every week knowing exactly what happened in the industry.

## Customization

Change the `industry` field in Set Industry Config to track any sector — SaaS tools, fintech, healthtech, etc. The workflow works for any industry.

## Setup

1. Perplexity API key
2. OpenAI API key
3. Notion integration + Industry Intelligence DB
4. Slack Bot Token + #market-intel channel
