# market-sizing-estimator

Market sizing done wrong is just confidence-inspiring fiction. "The market is $50B, so if we capture 1% that's $500M" is not a market size — it's a number designed to impress investors. Good market sizing uses both top-down and bottom-up approaches, quantifies uncertainty, explains every assumption, and is honest about what would make it larger or smaller. This searches for public market data via Tavily and builds a structured TAM/SAM/SOM model.

---

## What it does

Takes market description, product name, target customer, geographies, pricing model, and average revenue per unit. Searches Tavily for public market size data and industry reports. Claude produces:

- **Executive summary** — 3–4 sentences on market size, key opportunity, and the most important caveat
- **TAM** — Total Addressable Market with methodology (top-down/bottom-up/hybrid), rationale, and source references
- **SAM** — Serviceable Addressable Market with rationale for why this subset is serviceable
- **SOM** — Serviceable Obtainable Market with 3–5 year timeline rationale
- **Bottom-up model** — total addressable units × ARPU = implied TAM (sanity check against top-down)
- **Key assumptions** — each with confidence level (high/medium/low) and sensitivity analysis
- **Bull case / bear case** — SAM in optimistic and pessimistic scenarios with rationale
- **Competitive density** — fragmented/consolidated/duopoly/monopoly
- **Market growth rate**
- **Key risks** specific to this market
- **Data gaps** — what data would make this estimate more reliable

HTML report with TAM/SAM/SOM displayed as large numbers, bottom-up model, bull/bear case cards.

---

## Stack

n8n, Tavily API, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Setup

**Env vars:** `TAVILY_API_KEY`, `FROM_EMAIL`

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/estimate-market-size \
  -H "Content-Type: application/json" \
  -d '{
    "product_name": "Flowdesk",
    "market_description": "Project management software for operations teams at small and mid-market companies (10-200 employees)",
    "target_customer": "Operations managers and founders at companies with dedicated ops functions, frustrated with spreadsheets and overcomplicated enterprise PM tools",
    "geographies": ["US", "Canada", "UK"],
    "pricing_model": "SaaS subscription, $8-15 per user per month",
    "avg_revenue_per_unit": 1800,
    "currency": "USD",
    "competitor_context": "Asana, Monday.com, and Trello serve both SMB and enterprise. Notion is adjacent. No dominant player focused exclusively on ops-heavy teams at this size.",
    "assumptions": "Ops-heavy companies defined as those with 1+ dedicated ops role per 20 employees. Estimated 15% of companies 10-200 employees have dedicated ops function.",
    "business_goal": "Series A pitch",
    "reply_email": "ceo@flowdesk.com"
  }'
```

**Required:** `market_description`, `product_name`

---

## TAM vs SAM vs SOM

- **TAM** — everyone who could theoretically buy the product (all PM software buyers globally)
- **SAM** — the subset you can actually reach given your go-to-market, geography, and capabilities
- **SOM** — what you can realistically capture in 3–5 years given team size, funding, and competitive position

Claude always builds all three. Investors care about SAM and SOM — TAM is context.

---

## Bottom-up model

Claude always builds a bottom-up model as a sanity check against top-down numbers. Total units × ARPU = implied TAM. If the top-down TAM is $5B but the bottom-up model implies $800M, that's a signal the top-down number is inflated. Both models are shown side-by-side.

---

## Assumption sensitivity

Each key assumption includes a sensitivity note: "if this assumption is wrong by 2x, the SAM changes by approximately X%." This lets founders and investors understand which assumptions drive the model most and which are less consequential.

---

## Limitations

- Market sizing is inherently imprecise. The output is a structured estimate, not a prediction. Treat the numbers as directional.
- Tavily search finds publicly indexed market reports, not proprietary data. For a more defensible analysis, supplement with primary research.
- Bottom-up models require reasonable ARPU estimates. If `avg_revenue_per_unit` is not provided, Claude estimates from the pricing model — pass actual data for more accuracy.

---

## License

MIT.
