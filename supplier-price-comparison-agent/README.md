# FLOOWBOX - Supplier Price Comparison Agent

Procurement teams often stay with the same suppliers out of habit, leaving significant cost savings on the table. This workflow checks current market prices for every product weekly and flags where switching or negotiating would save money.

## What it does

Every Monday, fetches the product procurement list from Google Sheets. For each product, Perplexity searches IndiaMART, TradeIndia, Alibaba, and direct manufacturer sites for current wholesale prices with MOQ and lead times. GPT-4o compares all options against the current supplier price and recommends a specific action: switch now, get a sample first, stay with current, or use the data to negotiate. Calculates total monthly savings potential and posts a Slack report.

## Tools Used
- **Orchestration:** n8n
- **Market Research:** Perplexity AI Sonar (per product)
- **Comparison Engine:** OpenAI GPT-4o
- **Data:** Google Sheets
- **Report:** Slack
- **Schedule:** Weekly Monday 7 AM

## Flow

```
Monday 7 AM
  → Fetch product list from Google Sheets (max 8 per run)
  → For each product:
      → Perplexity: search wholesale prices (IndiaMART, Alibaba, etc.)
      → GPT-4o: compare + recommend action
  → Aggregate all comparisons
  → Calculate total monthly savings
  → Save to comparison log
  → Post Slack report
```

## Recommendation output per product

```json
{
  "product": "Cotton Tote Bags (100 units)",
  "current_price": 85,
  "best_supplier": "Dharwad Textiles, Karnataka",
  "best_price": 62,
  "savings_per_unit": 23,
  "monthly_savings": 2300,
  "all_options": [
    {"supplier": "Dharwad Textiles", "price_per_unit": 62, "moq": "50 units", "lead_time": "7 days", "location": "Karnataka"},
    {"supplier": "Tiruppur Exports", "price_per_unit": 68, "moq": "100 units", "lead_time": "10 days", "location": "Tamil Nadu"}
  ],
  "switch_recommended": true,
  "action": "get sample first",
  "risks": ["New supplier — quality verification needed before bulk order"]
}
```

## Google Sheets schema

| Column | Description |
|---|---|
| Product Name | Used for supplier search |
| Current Supplier | For comparison |
| Current Unit Price | Current cost |
| Monthly Volume | To calculate total savings |
| Quality Notes | Requirements to pass to comparison |

## Why I built this

A manufacturing client had not renegotiated supplier contracts in 2 years. First run of this workflow identified ₹28,000/month in potential savings across 6 products — just from checking current market rates. The "negotiate current supplier" recommendation alone paid for the automation setup.

## Setup

1. Google Sheets with procurement data
2. Perplexity API key
3. OpenAI API key
4. Slack Bot Token + #procurement channel
