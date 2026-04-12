# FLOOWBOX - Shopify Sales AI Weekly Report

Shopify gives you data. This workflow turns that data into decisions. Every Monday morning, a complete AI-analyzed sales report lands in your inbox and Slack — revenue trends, product performance, and specific recommended actions.

## What it does

Every Monday, fetches current week and previous week order data from the Shopify Admin API. Calculates revenue, order count, top products, and average order value for both periods. GPT-4o compares week-over-week, identifies top and underperforming products, flags inventory concerns, and generates specific promotion opportunities. Sends a formatted HTML email to the store owner and posts a Slack summary.

## Tools Used
- **Orchestration:** n8n
- **Data:** Shopify Admin REST API
- **Analysis:** OpenAI GPT-4o
- **Report:** Email (HTML)
- **Summary:** Slack
- **Schedule:** Weekly Monday 8 AM

## Flow

```
Monday 8 AM
  → Fetch this week orders (Shopify API)
  → Fetch last week orders (Shopify API)
  → Calculate metrics per period
  → GPT-4o: WoW comparison + insights + actions
  → Send email report
  → Post Slack summary
```

## Analysis output

```json
{
  "revenue_change_percent": 23,
  "performance": "strong",
  "recommended_actions": [
    "Restock 'Automation Handbook' — sold out 3x this week",
    "Run flash sale on slow-moving 'Starter Kit' — 0 units this week vs 8 last week",
    "Upsell opportunity: buyers of Product A also buy Product B 67% of the time"
  ],
  "promotion_opportunities": ["Bundle deal on top 2 products could increase AOV by ~15%"]
}
```

## Why I built this

A Shopify client was checking their analytics dashboard manually every Monday and writing a summary email by hand. This replaced that 45-minute weekly task with a fully automated report they receive before they even open their laptop.

## Setup

1. Shopify store Admin API token (Settings → Apps → Private apps)
2. Store domain (your-store.myshopify.com)
3. OpenAI API key
4. SMTP credentials
5. Slack Bot Token + #ecommerce channel
