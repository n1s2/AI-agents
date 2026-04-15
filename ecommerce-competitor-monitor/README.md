# FLOOWBOX - E-commerce Competitor Monitor

Knowing what competitors are doing before your customers do is a genuine edge. This workflow scans pricing, promotions, and customer sentiment across your competitors every morning — surfacing opportunities to exploit and threats to watch.

## What it does

Every morning, three Perplexity searches run simultaneously — one checking competitor pricing pages for any changes or discounts, one scanning for active promotions and new product launches, one monitoring customer reviews and social media sentiment for competitor weaknesses. GPT-4o synthesizes all three into a daily brief with specific actionable opportunities. High-urgency situations (price wars, major launches) trigger an immediate Slack alert.

## Tools Used
- **Orchestration:** n8n
- **Intelligence (x3):** Perplexity AI Sonar (parallel)
- **Analysis:** OpenAI GPT-4o
- **Alerts:** Slack (#competitor-intel)
- **Schedule:** Daily 7 AM

## Intelligence output

```json
{
  "pricing_changes": [{"competitor": "Zapier", "change": "Pro plan up 15%", "impact": "high"}],
  "competitor_weaknesses": ["Make.com users complaining about slow support this week"],
  "opportunities_to_exploit": [
    "Zapier price increase — reach out to price-sensitive leads with comparison",
    "Make.com support complaints — highlight FLOOWBOX response time in ads"
  ],
  "recommended_actions": ["Launch comparison landing page vs Zapier new pricing within 48 hours"]
}
```

## Why I built this

Missing a competitor's price increase by 2 weeks means missing the window to capture their unhappy customers. FLOOWBOX competes with Zapier and Make.com for automation clients — knowing when they stumble is directly relevant to sales conversations.

## Setup

1. Perplexity API key
2. OpenAI API key
3. Slack Bot Token + #competitor-intel channel
4. Update competitor names and product categories in Set Competitor Config
