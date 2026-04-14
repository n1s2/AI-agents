# FLOOWBOX - Refund Pattern Detector

A rising refund rate erodes margins silently. This workflow monitors refund patterns weekly, identifies root causes per product, and flags fraud signals — including customers gaming the refund system.

## What it does

Every Monday, fetches refund and order data from Google Sheets. Calculates refund rate, identifies top refunded products with their most common reasons, and flags customers with 3 or more refunds (serial refunders). GPT-4o analyzes the patterns to identify root causes — product quality issues, description mismatches, delivery problems — and fraud signals. High or medium fraud risk triggers an immediate Slack alert to #ecommerce-alerts.

## Tools Used
- **Orchestration:** n8n
- **Data:** Google Sheets (orders + refunds)
- **Pattern Analysis:** Code node (stats calculation)
- **Root Cause Analysis:** OpenAI GPT-4o
- **Alerts:** Slack (standard + urgent)
- **Schedule:** Weekly Monday 9 AM

## What gets calculated

- Overall refund rate vs industry benchmark (2-5%)
- Top 5 products by refund count with most common reason
- Top 5 refund reasons across all products
- Serial refunders (3+ refunds from same customer)

## Analysis output

```json
{
  "refund_rate_assessment": "elevated",
  "root_causes": [
    "Product images don't match actual color — 34% of refunds cite 'not as described'",
    "Fragile items damaged in transit — packaging issue"
  ],
  "product_issues": [
    {"product": "Blue Ceramic Mug", "likely_cause": "Color difference in photos vs product", "fix": "Retake photos in natural lighting"}
  ],
  "fraud_risk": "medium",
  "fraud_signals": ["3 customers have refunded 4+ times each, all claiming 'item not received'"],
  "immediate_actions": ["Block the 3 serial refund accounts", "Add delivery confirmation for orders >₹1000"]
}
```

## Refund data schema

**Refunds sheet:**
| Column | Type |
|---|---|
| Customer Email | Email |
| Product Name | Text |
| Refund Reason | Text |
| Refund Date | Date |
| Amount | Number |

## Why I built this

A client's refund rate crept from 3% to 8% over 6 months without anyone noticing because it was gradual. By the time it was caught manually, ₹45,000 in unnecessary refunds had been processed. This catches patterns in real time — a single week's data is enough to spot product-specific issues.

## Setup

1. Google Sheets with Orders + Refunds sheets
2. OpenAI API key
3. Slack Bot Token + #ecommerce and #ecommerce-alerts channels
