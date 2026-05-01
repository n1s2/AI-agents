# freelance-invoice-chaser

Chasing late invoices is the part of freelancing nobody talks about when they romanticize working for yourself. Most clients pay fine. A handful need a nudge. A few will make you send four emails over two months before they move.

I used to write these follow-ups manually, which meant either putting it off (awkward) or sending something too aggressive because I was annoyed. Neither is good.

This runs every morning, checks which invoices are overdue, and sends the right tone of follow-up based on how late the payment is. Stage 1 is a friendly reminder. Stage 4 is serious. Claude handles the wording so I don't have to think about it, and the sheet gets updated so the same invoice doesn't get chased twice in three days.

When something hits 30+ days overdue it also emails me a personal alert, because at that point the automation isn't enough and I need to decide whether to call, escalate, or write it off.

---

## What it does

- Runs daily at 9am
- Reads all invoices from a Google Sheet
- Skips paid, cancelled, and void invoices
- Skips invoices chased within the last 3 days (unless escalating)
- For each overdue invoice, determines the chase stage by days overdue:
  - Stage 1: 1–6 days → friendly reminder
  - Stage 2: 7–13 days → polite but clear
  - Stage 3: 14–29 days → firm and direct
  - Stage 4: 30+ days → formal, mentions further steps
- Claude writes a tailored email for each invoice + stage combo
- Sends the email from your address to the client
- Updates the sheet: last chased date, chase count, stage
- Stage 4 triggers a personal alert email to you

---

## Stack

- **n8n** — scheduling + workflow
- **Google Sheets** — invoice database
- **Anthropic Claude** (claude-opus-4-5) — email writing
- **SMTP** — sending emails

---

## Setup

### 1. Create the Invoices sheet

Make a Google Sheet called **Invoices** with these columns:

```
invoice_id | client_name | client_email | amount | currency | due_date | project_name | status | freelancer_name | notes | last_chased_at | chase_count | chase_stage
```

Fill in your invoices. Leave `last_chased_at`, `chase_count`, `chase_stage` blank — the workflow writes to those.

**Status values the workflow respects:** `paid`, `cancelled`, `void` → skipped. Everything else is treated as unpaid.

### 2. Environment variables

```
INVOICES_SHEET_ID=your_google_sheet_id
FREELANCER_EMAIL=you@yourdomain.com
FREELANCER_NAME=Your Name
```

### 3. Credentials

- **Google Sheets OAuth2** — n8n's Google setup
- **Anthropic API** — LangChain node
- **SMTP** — your email provider

### 4. Import and activate

Import `workflow.json`, set the env vars and credentials, activate. It'll run at 9am tomorrow. Test it first by clicking Execute Workflow with a test invoice row in your sheet.

---

## Marking invoices as paid

When a client pays, just update the `status` column to `paid`. The workflow skips it from that point on. You can do this directly in the sheet or build a small form — whatever works for your setup.

---

## The `notes` column

This is optional free text that Claude gets to see. Useful for context like:

- "Client said they'd pay end of month"
- "Long-term client, be extra gentle"
- "Disputed the late delivery — handle carefully"

Claude uses this to adjust the tone or acknowledge context in the email. It's the most underused column.

---

## Customizing the chase schedule

The stage thresholds are in the **Find Overdue Invoices** node:

```js
if (daysOverdue >= 30) stage = 4;
else if (daysOverdue >= 14) stage = 3;
else if (daysOverdue >= 7) stage = 2;
```

Change these to match your preferences. If you have net-60 terms and a gentler approach, you might push stage 3 to 30 days and stage 4 to 45.

The 3-day re-chase cooldown is also in the same node:

```js
if (daysSinceLastChase < 3 && stage === parseInt(row.chase_stage || 1)) continue;
```

Change 3 to whatever gap feels right for your business.

---

## Multiple currencies

The `currency` column handles this — just put `GBP`, `EUR`, `CAD`, etc. and it appears correctly in the email. The amount formatting is basic (no locale-specific separators), but it works for most cases.

---

## Using a custom email domain

For professional use, send from your actual business email. Gmail works for testing with an app password but clients can see the From address. If you use Fastmail, Zoho, or have a custom domain through Google Workspace, use those SMTP settings instead.

---

## What I don't automate

I don't automate Stage 4 beyond the email + personal alert. At 30+ days I want to decide manually whether to:
- Make a phone call
- Send a formal letter before action
- Involve a debt collection service
- Write it off as a loss (sometimes relationships matter more)

The alert gives me the signal to act — what I do is still my call.

---

## Limitations

- This sends one email per overdue invoice per run. If you have 10 overdue invoices you'll send 10 emails in one morning. Fine for most freelancers, could be surprising if you've been ignoring the sheet for a month.
- The workflow doesn't handle payment plans or partial payments. If a client paid half, mark the status as something custom and use the notes column to track it manually.
- Email deliverability depends on your SMTP setup. If you're sending from a personal Gmail and getting flagged as spam, look into SPF/DKIM records for your domain.

---

## Roadmap

- [ ] Stripe integration — auto-mark paid when Stripe webhook fires
- [ ] WhatsApp follow-up option for certain clients
- [ ] Monthly AR summary report
- [ ] CSV export of all outstanding invoices

---

## License

MIT.
