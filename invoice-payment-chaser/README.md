# invoice-payment-chaser

Late payments are one of the biggest cash flow problems for freelancers and small agencies. Most people either avoid chasing at all (too awkward) or send a generic "just checking in" that doesn't work. A well-written chase email — professional, specific, appropriately firm — gets paid faster than either.

This has two modes: a webhook to generate and optionally send a chase email for a specific invoice on demand, and a daily 9am automated check that scans a Google Sheet of invoices, identifies what's overdue, and sends escalating follow-ups automatically.

Tone escalates based on days overdue: gentle (≤7 days), firm (8–21), urgent (22–45), final notice (45+). You can override the tone manually.

---

## What it does

**On-demand webhook (POST `/chase-invoice`):**
- Takes invoice details: client, amount, due date, your name, project, payment instructions
- Auto-calculates days overdue and selects appropriate tone (or uses your override)
- Claude writes a complete chase email calibrated to tone — never apologetic, never aggressive
- Optionally sends the email directly to the client
- Logs the chase to Google Sheets
- Returns the email draft + suggested follow-up timing

**Daily automated check (9am):**
- Loads all invoices from the Invoices tab in Google Sheets
- Filters to overdue and unpaid
- For each: calculates tone based on days overdue, generates chase email, sends if `auto_chase` is true

---

## Stack

n8n (webhook + daily scheduler), Anthropic Claude (claude-sonnet-4-20250514), Google Sheets, SMTP.

---

## Setup

**Two Sheets tabs:**

**Invoices** — columns: `invoice_id | client_name | client_email | amount | currency | due_date | your_name | your_company | your_email | project_description | payment_instructions | status | previous_chase_count | auto_chase | include_late_fee | late_fee_amount | tone_override`

Set `status` to `paid` when payment is received to stop chasing. Set `auto_chase` to `true` to enable daily auto-send.

**ChaseLog** — columns: `chased_at | invoice_id | client_name | amount | days_overdue | tone | sent`

**Env vars:** `INVOICE_CHASE_SHEET_ID`, `FROM_EMAIL`

---

## Calling the webhook manually

```bash
curl -X POST https://your-n8n.com/webhook/chase-invoice \
  -H "Content-Type: application/json" \
  -d '{
    "invoice_id": "INV-2025-047",
    "client_name": "Brightfield Digital",
    "client_email": "accounts@brightfield.co",
    "amount": 4200,
    "currency": "USD",
    "due_date": "2025-05-01",
    "your_name": "Maya Osei",
    "your_company": "Maya Osei Design",
    "your_email": "maya@mayadesign.co",
    "project_description": "Brand identity redesign, delivered April 18",
    "payment_instructions": "Bank transfer to account ending 4821. Reference: INV-2025-047",
    "previous_chase_count": 1,
    "include_late_fee": true,
    "late_fee_amount": 63,
    "send_email": false
  }'
```

Pass `send_email: false` to get the draft without sending. Useful for reviewing before it goes out.

---

## Tone calibration

| Tone | Days overdue | Approach |
|---|---|---|
| `gentle` | 1–7 | Friendly reminder, assumes oversight, no consequences mentioned |
| `firm` | 8–21 | Direct, notes the delay is now notable, requests response |
| `urgent` | 22–45 | Clear, names consequences, may reference late fee |
| `final_notice` | 45+ | Last chance, specific deadline (3 business days), mentions collections/legal |

Claude writes each tone distinctly — a `firm` email doesn't say "I hope this finds you well" and a `gentle` email doesn't threaten legal action.

---

## Final notice emails

When tone is `final_notice`, the email:
- States clearly this is the final communication before escalation
- Gives a specific 3-business-day deadline
- Mentions collections agency or legal action as the next step
- Does not apologize, hedge, or leave an opening for further delays

---

## Late fees

Pass `include_late_fee: true` and `late_fee_amount` to include a late fee reference in the email. Claude incorporates this naturally — for firm/urgent tones as a mention, for final notice as a specific amount that has been added to the outstanding balance.

---

## Limitations

- The agent generates and optionally sends emails. It doesn't confirm delivery, track opens, or follow up on bounces — monitor your SMTP logs for delivery issues.
- `final_notice` emails should be reviewed before sending for large amounts or complex client relationships. The auto-chase feature is most reliable for amounts under $5,000 with clear payment terms in writing.
- The daily check processes all overdue unpaid invoices every day. To avoid chasing too frequently, add a `last_chased_date` column and filter to invoices where `last_chased_date` is more than N days ago in the Filter Overdue node.

---

## Ideas

- [ ] Escalation intervals: only chase every 3/7/14 days depending on tone level rather than daily
- [ ] Payment confirmation: a companion webhook that marks an invoice as paid and sends a receipt
- [ ] Accounting integration: pull invoices directly from Xero, QuickBooks, or FreshBooks rather than a manual sheet
- [ ] SMS chase: for final notice, send an SMS via Twilio in addition to the email

---

## License

MIT.
