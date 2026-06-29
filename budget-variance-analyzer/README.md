# budget-variance-analyzer

Budget vs actuals reports are usually a table of numbers that require a finance person to interpret. Department heads and business leaders don't always have time to dig into why payroll was $23k over or what the travel underspend means for Q4. This calculates variances, flags significant line items above your threshold, and gives a plain-English analysis: what's driving the variances, which are concerning vs explainable, what to investigate, and what the full-year forecast looks like if trends continue.

---

## What it does

1. Accepts: department, period, currency, significance threshold, and line items (each with category, budgeted, actual, optional notes)
2. Calculates variance and variance % per line item, flags significant ones (default: >10% variance)
3. Calculates totals and overall variance %
4. Claude analyzes:
   - Overall verdict: favorable / unfavorable / mixed / on target
   - 2–3 sentence assessment of the financial picture
   - Key drivers per significant line item (cause, whether it's a concern, whether it's one-time)
   - Concerns list and positive variances list
   - Recommended actions with urgency and owner
   - Forecast implication (if trend continues)
   - Questions for the department head to investigate
5. Builds a full HTML report with line-item table (color-coded), concerns/positives side-by-side, action plan
6. Emails to recipient list
7. Returns full JSON

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP.

---

## Setup

**Env vars:** `FROM_EMAIL`
**Credentials:** Anthropic API (LangChain node), SMTP.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/analyze-budget-variance \
  -H "Content-Type: application/json" \
  -d '{
    "department": "Engineering",
    "period": "Q1 2025",
    "currency": "USD",
    "significance_threshold": 10,
    "context": "Q1 was our biggest hiring quarter — we onboarded 6 engineers in February. We also ran two offsites that were planned but not fully reflected in the original budget.",
    "recipient_emails": ["cfo@company.com", "vp-eng@company.com"],
    "line_items": [
      {"category": "Salaries & Benefits", "budgeted": 420000, "actual": 487000, "notes": "6 new hires in Feb, some with signing bonuses"},
      {"category": "Contractors", "budgeted": 60000, "actual": 22000, "notes": "Reduced contractor usage due to FTEs hired"},
      {"category": "Software & Tools", "budgeted": 48000, "actual": 51200},
      {"category": "Travel & Offsites", "budgeted": 18000, "actual": 34500, "notes": "Two team offsites, one unplanned Q1 conference"},
      {"category": "Training & Development", "budgeted": 12000, "actual": 4800, "notes": "Most courses moved to Q2"},
      {"category": "Hardware", "budgeted": 25000, "actual": 28400},
      {"category": "Cloud Infrastructure", "budgeted": 55000, "actual": 58900}
    ],
    "reply_email": "finance@company.com"
  }'
```

**Required:** `department`, `period`, `line_items`

---

## Significance threshold

`significance_threshold` sets which line items get flagged for review (default 10%). Items with variance % above this threshold are marked with ⚠ in the table and become the focus of the Claude analysis. Set lower (5%) for tight financial periods, higher (15–20%) for departments with inherently variable spend.

---

## Context field

Pass any relevant context from the department head in `context`. Claude uses this to distinguish explainable variance (e.g., "salary overspend is because we hired ahead of schedule as noted") from concerning variance (e.g., "cloud infrastructure is 7% over with no context provided"). Without context, Claude treats all significant variance as potentially concerning.

---

## Line item notes

Each line item can have a `notes` field — useful for pre-explaining variances you already understand. These appear in the table and are used in the variance driver analysis.

---

## Limitations

- This is a period-in-isolation analysis — it doesn't maintain history or compare to prior periods unless you include prior period data in the `context` field.
- Forecast implication is qualitative, not quantitative — Claude reasons about the direction and scale, not a precise full-year number.
- For departments with complex accruals, prepayments, or inter-company charges, include this context in the `context` field so Claude can reason about it correctly.

---

## Ideas

- [ ] Google Sheets trigger: pull budget and actual from a Sheet automatically each month
- [ ] Multi-department rollup: submit multiple departments and generate a company-wide summary
- [ ] YTD tracking: include year-to-date actuals alongside the period actuals for trend context
- [ ] ERP integration: pull actuals directly from QuickBooks, Xero, or NetSuite via API

---

## License

MIT.
