# FLOOWBOX - Demographic Trend Analyzer

Your target customer is not static — their behaviors, preferences, and pain points shift month to month. This workflow tracks demographic trends, market size data, and buying behavior shifts every month to keep your ICP current and your messaging relevant.

## What it does

On the 1st of every month, three parallel searches gather demographic intelligence. First stream pulls behavioral and demographic trends from the target segment using recent research from NASSCOM, McKinsey, and similar sources. Second stream fetches current TAM/SAM data and growth rate projections for the product category. Third stream tracks how buying behaviors and decision-making are shifting in the target segment. GPT-4o synthesizes into an updated ICP, the best acquisition channels for this month, and messaging that currently resonates.

## Tools Used
- **Orchestration:** n8n
- **Research (x3):** Perplexity AI Sonar (parallel)
- **Intelligence:** OpenAI GPT-4o
- **Storage:** Notion
- **Strategy Brief:** Slack
- **Schedule:** Monthly on 1st

## Monthly intelligence output

```json
{
  "market_size_tam": "$2.8B (Indian B2B SaaS for SMEs)",
  "market_growth_rate": "28% CAGR through 2028",
  "segment_trends": [
    "Founders 30-40 now prefer video demos over text — 68% make decisions after a live walkthrough",
    "WhatsApp has become the dominant B2B communication channel for Indian SME owners"
  ],
  "best_acquisition_channels": ["LinkedIn (decision-makers)", "WhatsApp communities", "Founder events"],
  "messaging_that_resonates": [
    "'Save 10 hours per week' outperforms 'increase efficiency' 3x in ad testing",
    "ROI framing in rupees (not percentages) performs better with Indian SME segment"
  ]
}
```

## Why I built this

FLOOWBOX's ICP when we started was different from who actually converted 6 months in. The segment's behavior had shifted — WhatsApp replaced email as the primary outreach channel for Indian SMEs. This workflow would have caught that shift in real time instead of discovering it through failed campaigns.

## Setup

1. Perplexity API key
2. OpenAI API key
3. Notion integration + DB ID
4. Slack Bot Token + #strategy channel
5. Update target_segment and product_category in Set Analysis Config
