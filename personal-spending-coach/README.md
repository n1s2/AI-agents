# personal-spending-coach

Every budgeting app I've tried has the same problem: too much friction to log things, so after two weeks I stop. Or the automatic bank sync is there but it imports everything with useless merchant names and I spend more time cleaning data than understanding it.

This is two things in one: a dead-simple webhook to log expenses as they happen (I call it from a phone shortcut in about 10 seconds), and a Sunday morning email with a real spending analysis from Claude. Not a dashboard I have to open. Not a notification I'll ignore. Just an email that tells me where my money actually went and whether any of it warrants a second look.

The Sunday report compares this week to my monthly budget targets, flags over-budget categories with color-coded bars, and includes a coaching note from Claude that references my actual numbers — not generic advice.

---

## What it does

**Expense logging (webhook, any time):**
- Accepts a POST: amount, category, description, merchant, whether it's essential
- Validates and saves to Google Sheets
- Returns confirmation immediately

**Weekly report (every Sunday 8am):**
- Loads all expenses from Google Sheets
- Loads monthly budget targets from a second sheet tab
- Computes: week total, month-to-date, vs last month %, essential spend %, category breakdown, top merchants, biggest transactions
- Sends everything to Claude which writes a direct coaching note based on the actual numbers
- Builds a formatted HTML email with category bars (green/amber/red vs budget)
- Delivers to your inbox

---

## Stack

- **n8n** — webhook + weekly scheduler
- **Google Sheets** — expense log + budget targets
- **Anthropic Claude** (claude-opus-4-5) — coaching analysis
- **SMTP** — email delivery

---

## Setup

### 1. Create the Google Sheet

Two tabs:

**Tab: Expenses** — columns:
```
expense_date | amount | currency | category | description | merchant | is_essential | note | logged_at
```
Leave it empty — the webhook fills it.

**Tab: Budget** — columns:
```
category | monthly_budget
```
Fill in your targets:

| category | monthly_budget |
|---|---|
| food_dining | 300 |
| groceries | 400 |
| transport | 150 |
| entertainment | 100 |
| subscriptions | 80 |
| shopping | 200 |
| health | 100 |
| personal_care | 60 |
| other | 100 |

You don't need to set a budget for every category. Ones without a budget still appear in the report, just without a progress bar.

### 2. Environment variables

```
SPENDING_SHEET_ID=your_google_sheet_id
FROM_EMAIL=coach@yourdomain.com
USER_EMAIL=you@email.com
```

### 3. Credentials

- **Google Sheets OAuth2**
- **Anthropic API**
- **SMTP**

### 4. Import and activate

Import `workflow.json`, activate. The weekly scheduler fires every Sunday at 8am. Test the webhook immediately by calling it manually (see below).

---

## Logging an expense

```bash
curl -X POST https://your-n8n.com/webhook/log-expense \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 34.50,
    "currency": "USD",
    "category": "food_dining",
    "description": "Lunch with client",
    "merchant": "Shake Shack",
    "is_essential": false,
    "expense_date": "2025-04-29"
  }'
```

**Valid categories:**
`food_dining`, `groceries`, `transport`, `entertainment`, `shopping`, `health`, `subscriptions`, `utilities`, `travel`, `education`, `personal_care`, `gifts`, `other`

**Required fields:** `amount`, `category`, `description`

---

## iOS Shortcut for fast logging

On iPhone I use a Shortcut that pops up three text fields (amount, description, category picker), then POSTs to the webhook. Total time: under 10 seconds. This is what makes the system actually work — if logging takes 30 seconds I'll stop doing it.

If you want the Shortcut file, open an issue and I'll add it to the repo.

---

## The coaching note

Claude gets the raw numbers: week spend, month vs last month, every category's actual vs budget, top merchants, biggest individual transactions. It's told to be direct and specific — if you're 140% on food_dining it'll say that, not hedge it. It also notices things like "your three biggest transactions this week were all food delivery, which accounts for 60% of your food budget" rather than giving general advice about eating out less.

It won't moralize. The goal is awareness, not guilt.

---

## Currency

Set currency per transaction. The report uses whatever currency is in the most recent row. Multi-currency support is not built in — if you travel and mix currencies, normalize everything to one currency before logging or add a conversion step.

---

## The `is_essential` field

Mark groceries, utilities, rent, medication as essential. This lets the report show you what percentage of your spending is discretionary vs necessary — useful context for whether your budget is actually tight or just poorly distributed.

---

## Manual weekly report

Don't wait until Sunday. Execute the **Weekly Sunday Report** → **Load All Expenses** → ... path manually in n8n by clicking Execute Workflow and starting from that trigger node. Useful when you first set it up and want to test with existing data.

---

## Limitations

- No automatic bank import. This is intentional — manual logging means you actually think about what you're spending. If you want auto-import, you'd need a Plaid or Teller.io integration upstream.
- The month comparison is calendar month vs previous calendar month, not rolling 30 days. If you set this up mid-month, the first report will look odd.
- Claude's analysis is only as good as your data. If you log expenses vaguely ("stuff" × 12) the coaching note won't be useful. Merchant names and descriptions matter.

---

## Ideas I might add

- [ ] Expense query endpoint: `GET /spending-summary?period=this_month`
- [ ] Over-budget alert email mid-week when a category crosses 80%
- [ ] Year-in-review report each January
- [ ] Shared household mode (two people, one sheet, split by user_id)

---

## License

MIT.
