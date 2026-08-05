# customer-win-loss-analyzer

A CRM loss-reason field with 40 different free-text entries doesn't tell leadership anything actionable. This groups win/loss data into themes, separates controllable factors (product gaps, pricing) from uncontrollable ones (budget freeze, reorg), breaks down performance by competitor and segment, and produces specific recommendations tied to owners (sales/product/marketing/pricing).

---

## What it does

Takes up to 60 deals (won or lost) with company name, deal value, competitor, segment, rep, primary reason, and notes. Claude produces:

- **Win rate** and analysis summary
- **Loss themes** — grouped patterns with deal count, total value at stake, controllable flag, specific pattern description, and recommendation
- **Win themes** — what's working, why, and how to reinforce it
- **Competitor breakdown** — deals against each competitor, win rate against them, and pattern
- **Segment patterns** — win rate by segment with notable patterns
- **Controllable vs uncontrollable** — percentage split with interpretation
- **Product gaps surfaced** — specific gaps repeatedly mentioned in loss reasons
- **Pricing signals** — patterns around pricing objections
- **Recommended actions** — each with owner (sales/product/marketing/pricing), priority, and expected impact

HTML report with win rate prominently displayed, loss theme cards (controllable flagged in red), competitor and segment tables, and action list with owners.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/analyze-win-loss \
  -H "Content-Type: application/json" \
  -d '{
    "product_name": "Flowdesk",
    "period_label": "Q2 2025",
    "currency": "USD",
    "competitive_context": "Primary competitors are Asana (enterprise-leaning) and Monday.com (SMB-focused, similar price point)",
    "reply_email": "sales-ops@flowdesk.com",
    "deals": [
      {"id": "D-001", "company_name": "Beacon Logistics", "outcome": "won", "deal_value": 14400, "segment": "logistics", "rep_name": "Jake", "primary_reason": "Fast implementation, no IT needed", "notes": "Ops manager was the champion, closed in 3 weeks"},
      {"id": "D-002", "company_name": "TechStart Inc", "outcome": "lost", "deal_value": 9600, "competitor": "Asana", "segment": "tech_startup", "rep_name": "Jake", "primary_reason": "Dev team already used Asana", "notes": "Would have needed to convince engineering to switch, ops team wanted us but lost internal battle"},
      {"id": "D-003", "company_name": "Meridian Consulting", "outcome": "lost", "deal_value": 6000, "segment": "professional_services", "rep_name": "Sara", "primary_reason": "Budget freeze", "notes": "Loved the product, CFO froze all new software spend company-wide"},
      {"id": "D-004", "company_name": "Pacific Agency", "outcome": "won", "deal_value": 3600, "segment": "agency", "rep_name": "Sara", "primary_reason": "Simpler than alternatives", "notes": "Compared to Monday.com, chose us for simplicity and price"},
      {"id": "D-005", "company_name": "Northwind Freight", "outcome": "lost", "deal_value": 12000, "competitor": "Asana", "segment": "logistics", "rep_name": "Jake", "primary_reason": "Needed advanced reporting we do not have", "notes": "Explicitly said they needed custom dashboards and cross-project reporting, we do not have that yet"}
    ]
  }'
```

**Required:** `deals`

---

## Controllable vs uncontrollable

This is the key distinguishing feature — a loss to "budget freeze" isn't something sales or product can fix, but a loss to "missing feature X" is. Claude's `controllable_vs_uncontrollable` breakdown helps leadership understand how much of the loss rate is actually addressable versus market conditions outside the team's control.

---

## Themes, not individual reasons

Rather than listing 40 unique loss reasons, Claude groups them into actionable themes — "Dev team already used a competitor" might combine 5 individual deal notes into one theme with a specific recommendation ("Build integration marketing for teams with existing Asana usage in other departments").

---

## Owners on recommendations

Every recommended action has an owner: sales, product, marketing, or pricing. This makes the analysis directly usable in cross-functional planning — product gets a specific list of what feature gaps are costing deals, pricing gets specific objection patterns, marketing gets competitive positioning gaps.

---

## Limitations

- Analysis quality depends on note detail. "Lost on price" produces a thin theme. "Lost on price — prospect specifically compared our $15/seat to Monday's $12/seat and couldn't justify the premium without clearer ROI messaging" produces an actionable one.
- Up to 60 deals per call. For larger datasets, segment by quarter or product line and run separately.

---

## License

MIT.
