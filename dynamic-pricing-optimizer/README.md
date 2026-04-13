# FLOOWBOX - Dynamic Pricing Optimizer

Pricing too high loses sales. Pricing too low leaves money on the table. This workflow researches competitor prices daily and recommends optimal prices based on live market data, your cost structure, and sales velocity.

## What it does

Every morning at 6 AM, fetches the product list from Google Sheets. For each product, Perplexity searches for current competitor prices on Amazon, Flipkart, and other platforms. GPT-4o calculates the recommended optimal price factoring in cost price, target margin, competitor range, and current sales velocity. Only alerts on significant price changes (over 3% difference). Logs all recommendations to a pricing history sheet.

## Tools Used
- **Orchestration:** n8n
- **Competitor Research:** Perplexity AI Sonar (per product)
- **Pricing Engine:** OpenAI GPT-4o
- **Data:** Google Sheets (product list + log)
- **Alerts:** Slack (significant changes only)
- **Schedule:** Daily 6 AM

## Flow

```
6 AM daily
  → Fetch product list (Google Sheets)
  → For each product (max 10/run):
      → Perplexity: current competitor prices
      → GPT-4o: calculate optimal price
  → Filter: only changes > 3%
  → Log all recommendations
  → Slack: significant change alerts
```

## Pricing recommendation output

```json
{
  "current_price": 2999,
  "recommended_price": 2499,
  "price_change_direction": "decrease",
  "price_change_percent": 17,
  "reasoning": "Competitor median is ₹2,200. Current price is 36% above median causing sales velocity drop. Reducing to ₹2,499 maintains 28% margin while improving competitiveness.",
  "competitor_range": {"min": 1899, "max": 3299, "median": 2200},
  "expected_margin": 28,
  "risk": "low",
  "confidence": "high"
}
```

## Google Sheets schema

| Column | Description |
|---|---|
| Product Name | Product title (used for competitor search) |
| Current Price | Your current selling price |
| Cost Price | Your cost/landed price |
| Target Margin | Minimum acceptable margin % |
| Weekly Sales | Current units sold per week |

## Why I built this

A client in the electronics accessories space was pricing by gut feel — checking Amazon manually once a week. They were consistently 15-20% above the market on some products and underpriced on others. This workflow runs daily and catches pricing drift before it affects revenue.

## Setup

1. Google Sheets with product pricing data
2. Perplexity API key
3. OpenAI API key
4. Slack Bot Token + #pricing channel
