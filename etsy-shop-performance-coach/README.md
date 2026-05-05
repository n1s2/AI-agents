# etsy-shop-performance-coach

Running an Etsy shop is strange because you get a flood of numbers — views, visits, favorites, conversion rate — and no help interpreting them. I know someone who had 800 views on a listing one week and zero sales and genuinely didn't know whether that was a pricing problem, a photo problem, or just a slow week.

This takes your weekly Etsy stats, runs them through Claude, and sends you a Monday morning report with context: what changed vs last week, 4-week averages so a slow week looks slow in proportion, the top listings by views with their conversion rates color-coded, and a "getting clicks but not selling" section for listings that clearly have something wrong.

The coaching note is the part that's actually useful. It references your numbers specifically and gives one or two concrete things to look at this week — not "try better photos" but "your candles listing is converting at 0.4% against 800 views, which is low even for that price point."

Because Etsy doesn't have a public API with stats, this uses a Google Sheet as the data source. You copy your weekly numbers from the Etsy dashboard and either paste them into the sheet or POST them via the companion webhook.

---

## What it does

**Weekly report (every Monday 7am):**
- Loads your stats history and listing data from Google Sheets
- Computes: revenue, orders, visits, views, favorites, conversion rate, avg order value
- Compares this week vs last week (% change), tracks 4-week rolling averages
- Identifies top 5 listings by views with conversion rates
- Flags listings with 20+ views and zero sales ("cold listings")
- Claude writes a coaching note based on the real numbers
- Sends a formatted HTML email with stat cards, listing table, and cold listings section

**Manual stats upload (webhook):**
- POST a week's stats directly from your Etsy dashboard without opening the sheet
- Saves to Google Sheets and confirms

---

## Stack

- **n8n** — weekly scheduler + webhook
- **Google Sheets** — stats history + listing performance data
- **Anthropic Claude** (claude-opus-4-5) — coaching analysis
- **SMTP** — email delivery

---

## Setup

### 1. Create the Google Sheet

Two tabs:

**Tab: Weekly Stats** — columns:
```
week_ending | revenue | orders | visits | views | favorites | logged_at
```

Fill in your past few weeks manually from your Etsy stats dashboard to get the comparison working from day one.

**Tab: Listings** — columns:
```
listing_title | price | views_this_week | orders_this_week | days_listed
```

Update this weekly with your top listings' performance. You only need to track the listings you care about — 10-20 is plenty.

### 2. Environment variables

```
ETSY_SHEET_ID=your_google_sheet_id
SHOP_NAME=Maple & Thread Co
SHOP_OWNER_EMAIL=you@email.com
FROM_EMAIL=reports@yourdomain.com
```

### 3. Credentials

- **Google Sheets OAuth2**
- **Anthropic API** (LangChain node)
- **SMTP**

### 4. Import and activate

Import `workflow.json`, activate. Test by executing the Monday scheduler manually — you'll need at least 2 rows in the Weekly Stats sheet for the comparison to work.

---

## Entering stats each week

Etsy's dashboard doesn't push to external systems. You have to copy the numbers manually. Two options:

**Option A: Edit the sheet directly**
Open Weekly Stats, add a new row for the current week ending date. Takes 2 minutes.

**Option B: POST to the webhook**
```bash
curl -X POST https://your-n8n.com/webhook/etsy-stats \
  -H "Content-Type: application/json" \
  -d '{
    "week_ending": "2025-05-04",
    "revenue": 487.50,
    "orders": 12,
    "visits": 890,
    "views": 1240,
    "favorites": 34
  }'
```

Do this Sunday evening so the Monday report has current data.

---

## The Listings tab

Update this weekly too, but it only needs your tracked listings — not every listing in your shop. Focus on:
- Your best sellers
- New listings you're watching
- Any listing that seems underperforming

The "cold listings" detection looks for items with 20+ views and zero orders. If a listing consistently appears there, it's worth looking at pricing, photos, or the description.

---

## Conversion rate benchmarks

Etsy's average conversion rate is roughly 1-3%. Claude knows this and uses it when commenting on your numbers:
- Below 1%: something is probably wrong (pricing, photos, or mismatch between title and listing)
- 1-3%: normal range
- Above 3%: strong — don't mess with what's working

---

## The cold listings section

This is the most actionable part of the report. A listing getting 50 views and zero sales in a week is a signal. It means people are finding it (search is working) but not buying it. The typical culprits:
- Price is too high relative to what the photos show
- Main photo doesn't make the product look appealing on small screens
- Description doesn't answer the question "why this one specifically"
- Shipping costs are showing up as a surprise in checkout

Claude will sometimes identify what it thinks the likely issue is based on the price point — a $200 item converting at 0% is different from a $12 item at 0%.

---

## Favorites

Favorites are tracked but used lightly in the coaching note. A high favorites-to-orders ratio means people like the item but aren't buying — can indicate price sensitivity or "saving for later" behavior. Claude mentions this when the pattern is notable.

---

## Multi-shop

The workflow is single-shop. For multiple shops, duplicate the workflow and set different `ETSY_SHEET_ID` and `SHOP_NAME` env vars for each. You'd also want different sheet tabs or separate sheets per shop.

---

## Limitations

- No Etsy API integration. All data is manual entry. If Etsy ever opens up a proper API with stats access, this could be automated further.
- The listing data table requires weekly manual updates. If you don't update it, the listing analysis won't reflect the current week.
- The coaching note is only as insightful as the data. If you have two weeks of data it's limited — after 8-10 weeks of history it gets much more useful for spotting patterns.

---

## Ideas

- [ ] Seasonal comparison: this week vs same week last year
- [ ] Star review tracker: monitor new reviews and surface ones worth responding to
- [ ] Listing refresh reminder: flag listings that haven't sold in 60+ days
- [ ] Revenue goal tracker: set a monthly target, show progress toward it each week

---

## License

MIT.
