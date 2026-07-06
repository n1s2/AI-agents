# board-report-generator

Board reports are often either too long (exhaustive updates nobody reads) or too shallow (metrics without context). Board members want signal: what happened, what it means, what you need from them. This generates a structured board report from your metrics and narrative inputs — honest, context-rich, and formatted for time-constrained readers.

---

## What it does

Takes company name, reporting period, a metrics object, narrative context, key highlights, risks, outlook, and what you're asking from the board. Claude generates:

- **Executive summary**: 4–6 sentences — what happened, what it means, most important thing to know. Honest, not promotional.
- **Financial performance**: headline, narrative with context, key figures table (metric / value / vs prior / context)
- **Growth metrics**: same structure
- **Product highlights**: shipped items, next period focus
- **Team & operations**: headline and narrative
- **Risks & mitigations**: each with severity, mitigation, and status (open/mitigated/monitoring)
- **Ask of board**: each ask with why you need it
- **Management commentary**: candid perspective — what gives confidence, what keeps the CEO up at night

HTML output formatted as a professional board document with serif font, dark header, confidential footer, risk severity color-coding, and financial metrics tables.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-board-report \
  -H "Content-Type: application/json" \
  -d '{
    "company_name": "Flowdesk",
    "reporting_period": "Q1 2025",
    "prepared_by": "Maya Chen, CEO",
    "currency": "USD",
    "reply_email": "board-prep@flowdesk.com",
    "narrative_context": "Q1 was our strongest growth quarter to date, driven by the Notion integration launch in February which drove a significant inbound spike. However, we ended the quarter with higher burn than planned due to accelerated hiring ahead of the enterprise product launch scheduled for Q2.",
    "key_highlights": [
      "ARR grew 34% QoQ to $2.1M",
      "Notion integration drove 1,200 new signups in first 3 weeks",
      "Hired 4 engineers ahead of schedule, team now 18 FTE",
      "Net revenue retention reached 118% for first time"
    ],
    "key_risks": [
      "Enterprise launch delayed by 3 weeks due to SSO implementation complexity",
      "CAC increased 22% in Q1 as paid channels became more competitive",
      "One of two enterprise sales reps resigned — backfilling now"
    ],
    "outlook": "Q2 guidance: ARR $2.6M-$2.8M. Enterprise launch targeted for May 15. Expecting 15-20 enterprise deals in pipeline by end of Q2 based on current LOIs.",
    "ask_of_board": [
      "Introductions to enterprise CISOs in your networks for security review conversations",
      "Guidance on whether to raise bridge round in Q3 vs extend runway through efficiency"
    ],
    "metrics": {
      "arr": {"value": 2100000, "prior": 1570000, "unit": "USD"},
      "mrr": {"value": 175000, "prior": 130800, "unit": "USD"},
      "customers": {"value": 4200, "prior": 3100},
      "nrr": {"value": 118, "prior": 106, "unit": "percent"},
      "gross_margin": {"value": 74, "unit": "percent"},
      "burn_rate": {"value": 380000, "prior": 280000, "unit": "USD/month"},
      "runway_months": {"value": 14},
      "headcount": {"value": 18, "prior": 14},
      "cac": {"value": 340, "prior": 278, "unit": "USD"},
      "ltv_cac": {"value": 4.2, "prior": 5.1},
      "new_customers_q1": {"value": 1100}
    }
  }'
```

**Required:** `company_name`, `reporting_period`, `metrics`

---

## Metrics format

Pass metrics as a free-form object — Claude handles any structure. Include `value`, `prior` (for comparison), and `unit` where relevant. Common fields: ARR, MRR, customers, NRR, gross margin, burn rate, runway, headcount, CAC, LTV/CAC.

---

## Honesty by design

Claude is explicitly instructed that boards hate surprises later and that good board reports are honest about challenges. The management commentary section asks for what keeps the CEO up at night, not just the good news. Risk mitigations are required for each risk — not just the risk itself.

---

## Limitations

- The report narrative is based on what you provide. If your metrics and context are sparse, the narrative will be thin. The richer the `narrative_context`, the stronger the report.
- The HTML is formatted for email or PDF print — not a slide deck. For a slide format, use this output as source material.

---

## License

MIT.
