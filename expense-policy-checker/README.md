# expense-policy-checker

Manual expense review takes a manager 10–20 minutes per report and applies policy inconsistently. The same $180 dinner gets approved one week and flagged the next depending on who reviews it. This applies your expense policy programmatically to each line item — per-item status (approved/flagged/rejected/needs_receipt/policy_ambiguous), specific policy reference, and a clear recommendation — then gives the approver a final verdict and exactly what to tell the submitter.

---

## What it does

Takes an array of expense items (category, description, amount, receipt flag, vendor, notes) and your expense policy (either as a structured object with limits per category or as a free-text policy document). Claude checks each item against the policy and returns:

- Overall verdict: approved / approved_with_exceptions / requires_review / rejected
- Per-line item: status, which policy rule applies, specific issue, recommendation (approve/reject/request_receipt/request_clarification)
- Policy violations: grouped violations with severity (hard_block/warning/information)
- Approved total and flagged total
- Missing information needed to make a final determination
- What the approver should tell the submitter (specific, actionable)

HTML report with line item table, violation cards, and verdict badge. Emails to approver and/or reply email.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/check-expense \
  -H "Content-Type: application/json" \
  -d '{
    "submitter_name": "Tom Walsh",
    "submitter_email": "tom@company.com",
    "submission_date": "2025-05-30",
    "period": "May 2025",
    "currency": "USD",
    "approver_email": "finance@company.com",
    "policy": {
      "text": "Meals: up to $75/person for client meals, $40/person for team meals, receipts required over $25. Travel: economy class only, hotel up to $250/night in major cities $350/night, receipts required. Software/subscriptions: pre-approved list only, otherwise requires manager approval before purchase. Entertainment: up to $150/person with prior manager approval. No personal expenses reimbursable."
    },
    "expense_items": [
      {"id": "EXP-001", "category": "meals", "description": "Team lunch — 4 people", "amount": 186, "date": "2025-05-12", "vendor": "Nobu", "receipt": true, "notes": "Sprint planning team lunch"},
      {"id": "EXP-002", "category": "software", "description": "Linear annual subscription", "amount": 96, "date": "2025-05-15", "vendor": "Linear", "receipt": true},
      {"id": "EXP-003", "category": "travel", "description": "Flight NYC to SF, economy", "amount": 340, "date": "2025-05-18", "vendor": "Delta", "receipt": true},
      {"id": "EXP-004", "category": "meals", "description": "Client dinner — Beacon Logistics (2 people)", "amount": 210, "date": "2025-05-20", "vendor": "Smith & Wollensky", "receipt": false},
      {"id": "EXP-005", "category": "other", "description": "Uber home from office at 11pm", "amount": 34, "date": "2025-05-22", "vendor": "Uber", "receipt": true}
    ]
  }'
```

**Required:** `expense_items`, `policy`

---

## Policy format

Pass policy as either:
- **Structured object**: `{"meals": {"per_person_limit": 75, "team_meal_limit": 40, "receipt_required_above": 25}, "travel": {"hotel_limit": 250}}`
- **Free text**: `{"text": "Meals: up to $75/person..."}`

Free text works well for most company policies — paste your existing policy text directly.

---

## Item status definitions

| Status | Meaning |
|---|---|
| `approved` | Compliant with policy, no action needed |
| `flagged` | Exceeds limits or has issues — approver must decide |
| `rejected` | Clear policy violation — should not be approved |
| `needs_receipt` | Amount requires receipt, none attached |
| `policy_ambiguous` | Cannot determine compliance without more info or policy clarification |

---

## Limitations

- This applies policy as Claude understands it from your policy text. Test with known cases before deploying in production. Complex multi-rule policies with exceptions may need additional clarity.
- Claude cannot verify that receipts are authentic, that vendors are real, or that the description matches the receipt — it checks declared information against policy rules.

---

## License

MIT.
