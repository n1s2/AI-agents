# inventory-reorder-alert

Running out of stock costs more than the lost sale. You lose the customer to a competitor, you lose ranking if you're on a marketplace, and you pay expedited shipping to recover. Most inventory problems are predictable — the reorder point was known, the lead time was known, someone just didn't check in time.

This runs every morning, loads your inventory from Google Sheets, identifies everything below its reorder point or at risk of stockout before the next shipment arrives, and sends the operations team a prioritized alert with the items ranked by urgency — out of stock, critical (less than 2 days remaining), reorder now, and watch. Claude reads the full picture and adds a short note on what needs immediate action and whether there are any supplier-level patterns.

There's also a companion webhook for updating stock levels on the fly — useful after a delivery arrives or after an inventory count.

---

## What it does

**Daily scan (every day 8am):**
- Loads all SKUs from Google Sheets
- For each item: compares current stock to reorder point, calculates days of stock remaining based on avg daily sales, checks if stockout will occur within lead time
- Classifies urgency: `out_of_stock`, `critical`, `reorder_now`, `watch`
- Sends to Claude which writes a short ops note: what needs immediate action, supplier patterns, one operational observation
- Emails a formatted alert table to the ops team — only when items need attention
- Silent if everything is adequately stocked

**Stock level update (webhook `/update-stock`):**
- POST a SKU and new stock count
- Updates the Google Sheet immediately
- Returns confirmation

---

## Stack

- **n8n** — daily scheduler + webhook
- **Google Sheets** — inventory database
- **Anthropic Claude** (claude-sonnet-4-20250514) — alert analysis
- **SMTP** — email delivery

---

## Setup

### 1. Create the Inventory sheet

One tab: **Inventory** — columns:

```
sku | product_name | category | current_stock | reorder_point | reorder_qty | lead_time_days | avg_daily_sales | unit_cost | supplier_name | supplier_email | currency | last_ordered_date | last_updated | notes
```

Fill in your product catalog. The key columns for the alert logic:
- `current_stock` — current units on hand
- `reorder_point` — units at which to reorder (your safety stock + lead time demand)
- `reorder_qty` — how many units to order when reordering
- `lead_time_days` — days from order to receipt
- `avg_daily_sales` — average units sold per day (used to calculate days of stock remaining)
- `unit_cost` — cost per unit (used to calculate reorder cost)

### 2. Environment variables

```
INVENTORY_SHEET_ID=your_google_sheet_id
FROM_EMAIL=ops@yourcompany.com
OPS_EMAIL=operations@yourcompany.com
```

### 3. Credentials

- **Google Sheets OAuth2**
- **Anthropic API** (LangChain node)
- **SMTP**

### 4. Import and activate

Import `workflow.json`, activate. Test by running the daily scan manually.

---

## Urgency classification

| Status | Condition |
|---|---|
| `OUT OF STOCK` | `current_stock == 0` |
| `CRITICAL` | Days of stock remaining ≤ 2 |
| `REORDER NOW` | Below reorder point but not yet critical |
| `WATCH` | Approaching reorder point (within lead time) |

Only items with an urgency level are included in the alert. Items above their reorder point with adequate stock don't appear.

---

## Days of stock calculation

```
days_of_stock_left = current_stock / avg_daily_sales
```

If `avg_daily_sales` is 0 or blank, days remaining shows as `—` and stockout risk is assessed based on reorder point alone.

The stockout risk flag fires when `days_of_stock_left <= lead_time_days` — meaning you'd run out before a new order could arrive.

---

## Reorder cost

```
reorder_cost = reorder_qty × unit_cost
```

The alert email shows both per-item reorder cost and the total estimated reorder cost across all flagged items. Useful for ops managers who need to authorize a purchase order quickly.

---

## Updating stock levels

```bash
curl -X POST https://your-n8n.com/webhook/update-stock \
  -H "Content-Type: application/json" \
  -d '{
    "sku": "WH-BLUE-L",
    "current_stock": 145,
    "notes": "Received shipment from Pacific Goods 2025-05-08"
  }'
```

The update goes directly to the Google Sheet. The next morning's scan will reflect the new count. For same-day recalculation, trigger the daily scan node manually after updating.

---

## Setting reorder points

A good reorder point accounts for:
- **Lead time demand**: `avg_daily_sales × lead_time_days`
- **Safety stock**: buffer for demand variability and supplier delays

Simple formula:
```
reorder_point = (avg_daily_sales × lead_time_days) + safety_stock
```

A product selling 10 units/day with a 7-day lead time and 20 units safety stock gets a reorder point of 90.

---

## Supplier patterns in the Claude note

If multiple items from the same supplier are flagged on the same day, Claude notices and calls it out — "Three items from Pacific Goods are below reorder point, suggesting either a slow recent shipment or a demand spike." This is the kind of pattern that's obvious in the data but easy to miss when reviewing a list of 20 items.

---

## Alert frequency

The email only sends when there are items needing attention. If your inventory is healthy, no email. If you want a daily summary even when everything is fine, add a final "Send Summary" branch from the **Any Alerts?** node's false path.

---

## Limitations

- Stock counts must be kept current. The alert is only as accurate as your `current_stock` column. If you're not updating it daily (via the webhook, a sync from your POS/WMS, or manual entry), the scan will be based on stale data.
- `avg_daily_sales` is a static number in the sheet. It doesn't automatically account for seasonality or recent trend changes. Update it periodically or build a separate node that recalculates it from a sales history sheet.
- The workflow shows up to 25 items in the alert email table. If you have more, they're all in the computed data — just the display is capped.

---

## Ideas

- [ ] Automatic purchase order draft: when reorder is triggered, generate a draft PO email to the supplier
- [ ] Shopify/WooCommerce sync: pull current stock levels directly from your store instead of relying on manual updates
- [ ] Demand forecasting: replace static `avg_daily_sales` with a rolling 30-day average calculated from a sales history sheet
- [ ] Supplier lead time tracking: log actual vs estimated lead times, flag suppliers who are consistently late

---

## License

MIT.
