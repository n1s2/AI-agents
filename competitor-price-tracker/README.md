# competitor-price-tracker

Pricing intelligence is one of those things companies say they do but usually don't. Someone checks a competitor's website once a quarter, makes a note, and moves on. By the time a competitor drops prices significantly — a promotional campaign, a market repositioning, a response to their own cost pressures — your team finds out from a sales call gone wrong.

This runs daily, scans the web for current pricing mentions across your competitor list, extracts price data from search results, tracks changes over time, and immediately alerts when a significant price drop is detected. All results log to Google Sheets with a full history.

It uses web search to extract prices, which means it works without direct API integrations to competitor sites — if the price appears anywhere on the indexed web (product pages, review sites, pricing comparison tools, press releases), it gets picked up.

---

## What it does

**Daily scan (7am):**
- Loads all enabled competitors from Google Sheets
- For each: searches for current pricing using the configured query
- Extracts price values using pattern matching across all results
- Takes the median of extracted prices as the best estimate
- Compares to last known price — calculates change % and absolute change
- Logs to Price History tab
- Updates last known price in Competitors tab
- If change exceeds `price_drop_threshold_pct`: sends email alert immediately

**Manual trigger (webhook `/track-prices`):**
- POST to run a fresh scan immediately
- Optional: filter to a specific competitor

---

## Stack

- **n8n** — daily scheduler + webhook
- **Tavily API** — pricing search
- **Google Sheets** — competitor config + price history
- **SMTP** — drop alerts

---

## Setup

### 1. Create the Google Sheet

Two tabs:

**Tab: Competitors** — columns:
```
competitor_name | product_name | search_query | our_price | currency | category | alert_email | price_drop_threshold_pct | last_known_price | last_checked | enabled
```

**Tab: Price History** — columns:
```
checked_at | competitor_name | product_name | estimated_price | price_change | price_change_pct | vs_our_price_pct | significant_drop
```

Key fields:
- `search_query` — the exact query used to find pricing (e.g. `"Acme SaaS Pro plan pricing"` or `"Acme Widget price site:acmecorp.com"`) — defaults to `competitor_name product_name price` if blank
- `price_drop_threshold_pct` — alert when price drops by more than this % (default 10)
- `last_known_price` — seed with current known price; gets updated on each scan

### 2. Environment variables

```
PRICE_SHEET_ID=your_google_sheet_id
TAVILY_API_KEY=tvly-your-key
FROM_EMAIL=pricing@yourcompany.com
DEFAULT_ALERT_EMAIL=product@yourcompany.com
```

### 3. Credentials

- **Google Sheets OAuth2**
- **Anthropic API** (not used in this workflow — pure extraction logic)
- **SMTP**

### 4. Import and activate

Import `workflow.json`, activate. Test by running the daily scanner manually with one competitor configured.

---

## Writing good search queries

The `search_query` column is the most important field for getting accurate prices. Tips:

**For SaaS products:**
```
Acme CRM pricing plans 2025
Acme CRM Pro plan monthly cost
```

**For physical products:**
```
Acme Widget model X price buy
Acme Widget X Amazon
```

**For services:**
```
Acme Consulting hourly rate
Acme Agency pricing packages
```

The more specific the query, the better. Include the product tier name if you're tracking a specific plan. If the competitor has a pricing page, add `site:competitordomain.com pricing` — Tavily handles site-specific searches.

---

## How price extraction works

The workflow uses regex patterns to find monetary values in search result text:
- `$[number]` — USD prices
- `£[number]` — GBP prices
- `€[number]` — EUR prices
- `[number] per month` / `[number]/mo` — subscription prices

It extracts all matches, filters to a reasonable range (>0, <100,000), and takes the median value as the best estimate. The median is more robust than the mean — it ignores outliers like "save $500" or "$0.99 shipping."

**Important:** This extracts prices from text, not live database queries. Prices from third-party review sites, blog posts, and comparison tools may lag the source by days or weeks. For real-time pricing of publicly listed products, build a direct scraper instead.

---

## Alert threshold

`price_drop_threshold_pct` sets when alerts fire. Default is 10% — a 10% price drop is typically significant enough to warrant a response.

For fast-moving markets (SaaS, consumer electronics) where small changes matter, lower to 5%. For stable markets, raise to 15–20% to reduce noise.

---

## `vs_our_price_pct` field

When `our_price` is filled in, the price history includes `vs_our_price_pct` — how the competitor's price compares to yours as a percentage. A -15% means the competitor is 15% cheaper than you. Useful for tracking competitive positioning over time.

---

## Multiple tiers

To track multiple pricing tiers for the same competitor, add one row per tier:

```
Acme CRM | Starter plan | Acme CRM Starter plan price | 29
Acme CRM | Pro plan     | Acme CRM Pro plan price      | 79
Acme CRM | Enterprise   | Acme CRM Enterprise pricing  | null
```

---

## Manual trigger

```bash
curl -X POST https://your-n8n.com/webhook/track-prices \
  -H "Content-Type: application/json" \
  -d '{}'
```

For a specific competitor:
```bash
curl -X POST https://your-n8n.com/webhook/track-prices \
  -H "Content-Type: application/json" \
  -d '{ "competitor_name": "Acme Corp" }'
```

---

## Limitations

- Price extraction is probabilistic. If a competitor doesn't publicly list prices (enterprise pricing, "contact us" models), the workflow will return no price or irrelevant numbers. This is a hard limit — there's no workaround without direct access to their pricing data.
- Prices extracted from third-party sites may be stale. If a competitor dropped prices yesterday, it may take a day or two for comparison sites and blogs to reflect it. For products on Amazon or other marketplaces, consider adding direct URL monitoring.
- The extraction takes the median of found prices — this works well most of the time but can be wrong when there are few price mentions or when prices vary significantly across product configurations.

---

## Ideas

- [ ] Weekly pricing digest: summarize the week's price movements across all competitors in a single email
- [ ] Chart generation: visualize price history over time per competitor from the Price History sheet
- [ ] Price response workflow: when a significant drop is detected, auto-draft a pricing review memo for leadership
- [ ] Promotional detection: flag when prices look like limited-time promotions vs permanent changes (shorter window search)

---

## License

MIT.
