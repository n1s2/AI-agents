# sales-commission-calculator

Commission disputes usually come down to reps not understanding or trusting how their number was calculated. This does the actual math deterministically in code (not left to an LLM to compute), applies accelerator tiers based on quota attainment, then has Claude explain the calculation in plain language, flag anomalies worth double-checking, and produce a verification checklist for payroll before payout.

---

## What it does

Takes rep name, period, list of deals (amount, deal type, close date), commission plan (base rate, quota, accelerator tiers), and currency. The validator does deterministic calculation:

1. Sums total bookings across deals
2. Calculates quota attainment percentage
3. Determines which accelerator tier applies based on attainment thresholds
4. Calculates per-deal commission using the applicable rate and any deal-type multiplier
5. Sums total commission

Claude then reviews the calculation and produces:
- **Calculation summary** — plain language explanation of the total
- **Attainment narrative** — honest but encouraging performance summary
- **Anomalies flagged** — anything that looks like a plan misconfiguration or data quality issue, with recommendation
- **Breakdown explanation** — clear walkthrough of how the accelerator tier was determined
- **Verification checklist** — things payroll/finance should double-check before payout

HTML statement with total commission prominently displayed, attainment percentage, deal-by-deal breakdown table, and anomaly flags.

---

## Stack

n8n (Code node for deterministic math), Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/calculate-commission \
  -H "Content-Type: application/json" \
  -d '{
    "rep_name": "Jake Reyes",
    "period": "Q2 2025",
    "currency": "USD",
    "reply_email": "jake@flowdesk.com",
    "commission_plan": {
      "base_rate": 0.08,
      "quota": 150000,
      "accelerators": [
        {"attainment_threshold_pct": 100, "rate": 0.10},
        {"attainment_threshold_pct": 120, "rate": 0.13},
        {"attainment_threshold_pct": 150, "rate": 0.16}
      ]
    },
    "deals": [
      {"id": "D-201", "name": "Beacon Logistics", "amount": 14400, "deal_type": "new_business", "deal_type_multiplier": 1, "close_date": "2025-04-12"},
      {"id": "D-202", "name": "Meridian Transport - expansion", "amount": 6000, "deal_type": "expansion", "deal_type_multiplier": 0.5, "close_date": "2025-05-03"},
      {"id": "D-203", "name": "Pacific Agency", "amount": 3600, "deal_type": "new_business", "deal_type_multiplier": 1, "close_date": "2025-05-20"},
      {"id": "D-204", "name": "Northwind Freight", "amount": 96000, "deal_type": "new_business", "deal_type_multiplier": 1, "close_date": "2025-06-15"},
      {"id": "D-205", "name": "TechStart Inc", "amount": 45600, "deal_type": "new_business", "deal_type_multiplier": 1, "close_date": "2025-06-28"}
    ]
  }'
```

**Required:** `rep_name`, `deals`, `commission_plan`

---

## Commission plan format

```json
{
  "base_rate": 0.08,
  "quota": 150000,
  "accelerators": [
    {"attainment_threshold_pct": 100, "rate": 0.10},
    {"attainment_threshold_pct": 120, "rate": 0.13}
  ]
}
```

The highest threshold the rep's attainment meets or exceeds determines the applicable rate for all commissionable bookings in the period — a standard "cliff" accelerator structure. Adjust the calculation logic in the "Validate & Calculate" node if your plan uses marginal/tiered rates instead.

---

## Deal type multipliers

Pass `deal_type_multiplier` per deal to handle plans that pay differently for new business vs expansion vs renewal (e.g., 100% commission on new business, 50% on expansions). Default multiplier is 1.0 if not specified.

---

## Deterministic math, LLM explanation

The actual dollar calculations happen in a Code node using real arithmetic — not generated or estimated by the LLM. Claude's role is explaining the result and catching things that look off (a deal with an unusually large multiplier, an attainment percentage that seems inconsistent with the accelerator applied), not computing the numbers themselves. This is the right division of labor for anything involving actual payroll.

---

## Limitations

- This implements a cliff-accelerator model (single rate applied to all bookings based on total attainment). For marginal/tiered rate plans (different rates for different portions of bookings), the calculation logic needs adjustment.
- Always have finance/payroll verify calculations before actual payout — this is a calculation and explanation tool, not a payroll system of record.

---

## License

MIT.
