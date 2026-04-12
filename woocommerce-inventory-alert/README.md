# FLOOWBOX - WooCommerce Inventory Alert Agent

Running out of stock silently kills revenue. This workflow monitors inventory daily, calculates sales velocity, and alerts with exact reorder quantities before stockouts happen — not after.

## What it does

Every morning, fetches all in-stock products and the last 30 days of completed orders from WooCommerce. Calculates daily sales velocity per product. Flags items that are below the low-stock threshold AND items that will run out before the reorder lead time passes. GPT-4o calculates recommended reorder quantities based on velocity and sends email + Slack alerts only when action is actually needed.

## Tools Used
- **Orchestration:** n8n
- **Data:** WooCommerce REST API
- **Analysis:** OpenAI GPT-4o (reorder calculations)
- **Alert:** Email + Slack
- **Schedule:** Daily 7 AM

## Flow

```
7 AM daily
  → Fetch all in-stock products (WooCommerce API)
  → Fetch last 30 days completed orders
  → Calculate daily velocity per product
  → Flag: stock ≤ threshold OR days remaining ≤ lead time
  → IF any alerts:
      → GPT-4o: calculate reorder quantities
      → Send email alert
      → Slack notification
  → IF no alerts: silent pass
```

## Smart velocity-based reordering

Standard low-stock alerts fire at a fixed number. This workflow calculates actual sales velocity — so a product selling 10 units/day at 5 units remaining is flagged as critical, while a product with 5 units and 0 sales velocity is not flagged at all.

## Alert output

```json
{
  "critical_items": [
    {"name": "Automation Bundle", "current_stock": 3, "reorder_qty": 60, "stockout_in_days": 2}
  ],
  "upcoming_items": [
    {"name": "Starter Kit", "current_stock": 18, "reorder_qty": 40, "order_by_date": "Apr 20"}
  ]
}
```

## Why I built this

A WooCommerce client had a bestseller go out of stock over a weekend — lost ₹85,000 in sales. The manual inventory check was weekly. This runs daily and would have caught it 10 days earlier with an exact reorder quantity.

## Setup

1. WooCommerce REST API credentials (Consumer Key + Secret from WooCommerce settings)
2. Store URL
3. OpenAI API key
4. SMTP credentials
5. Slack Bot Token + #ecommerce channel
