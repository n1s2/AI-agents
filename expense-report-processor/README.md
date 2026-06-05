# expense-report-processor

Expense report processing is one of those finance workflows that eats time out of proportion to its value. Someone submits a report, a manager glances at it, it goes to finance, someone chases missing receipts, it comes back, eventually it gets approved. Most of the friction is avoidable — policy violations are predictable, missing receipts are identifiable upfront, and the review note is almost always the same structure.

This processes submitted expense reports automatically: validates each line item against configurable policy limits, flags violations with specific line references and explanations, generates an approval recommendation, and routes the HTML report simultaneously to the submitter and their manager. Finance gets a consistent format every time; submitters get instant feedback on what needs fixing.

---

## What it does

1. Accepts a POST: submitter details, period, list of expense line items (each with amount, category, date, merchant, receipt status, project code)
2. Policy check against configurable limits:
   - Missing receipts above threshold (default $25)
   - High-value single items above limit (default $500)
   - Meals over per diem (default $75)
   - Entertainment over limit (default $150)
   - Missing dates
3. Generates recommendation: `approve`, `pending_receipts`, `flagged_for_review`, `review_required`
4. Claude writes a brief reviewer note explaining the report and any flags
5. Logs to Google Sheets
6. Emails formatted HTML report to submitter
7. If `manager_email` provided: emails manager with approval recommendation
8. Returns full JSON including all flags and line items

---

## Stack

- **n8n** — webhook + workflow
- **Google Sheets** — expense report log
- **Anthropic Claude** (claude-sonnet-4-20250514) — review note
- **SMTP** — submitter confirmation + manager notification

---

## Setup

### 1. Create the Google Sheet

One tab: **Reports** — columns:
```
report_id | submitted_at | submitter_name | submitter_email | department | period | total_amount | currency | item_count | flag_count | recommendation | status
```

### 2. Environment variables

```
EXPENSE_SHEET_ID=your_google_sheet_id
FROM_EMAIL=expenses@yourcompany.com
```

### 3. Credentials

- **Google Sheets OAuth2**
- **Anthropic API** (LangChain node)
- **SMTP**

### 4. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/submit-expense \
  -H "Content-Type: application/json" \
  -d '{
    "submitter_name": "Alex Rivera",
    "submitter_email": "alex@company.com",
    "submitter_department": "Sales",
    "submitter_manager": "Sarah Chen",
    "manager_email": "sarah@company.com",
    "period": "May 2025",
    "currency": "USD",
    "purpose": "Client entertainment and travel for Q2 pipeline meetings in NYC",
    "default_project_code": "SALES-Q2",
    "policy_limits": {
      "meal_per_diem": 80,
      "entertainment_limit": 200,
      "receipt_required_above": 25,
      "max_single_item": 500
    },
    "expense_items": [
      {
        "date": "2025-05-06",
        "description": "Flight NYC round trip",
        "category": "travel",
        "amount": 387.00,
        "merchant": "Delta Airlines",
        "receipt_provided": true,
        "project_code": "SALES-Q2"
      },
      {
        "date": "2025-05-06",
        "description": "Hotel 2 nights",
        "category": "accommodation",
        "amount": 459.00,
        "merchant": "Marriott Midtown",
        "receipt_provided": true
      },
      {
        "date": "2025-05-07",
        "description": "Client dinner with Apex team (4 people)",
        "category": "entertainment",
        "amount": 284.00,
        "merchant": "Nobu NYC",
        "receipt_provided": true,
        "notes": "Attendees: Alex Rivera, Sarah Chen (remote), John Park (Apex), Lisa Wang (Apex)"
      },
      {
        "date": "2025-05-07",
        "description": "Taxi to dinner",
        "category": "travel",
        "amount": 22.50,
        "merchant": "Uber",
        "receipt_provided": false
      },
      {
        "date": "2025-05-08",
        "description": "Working lunch",
        "category": "meals",
        "amount": 34.00,
        "merchant": "Sweetgreen",
        "receipt_provided": true
      }
    ]
  }'
```

**Required:** `submitter_name`, `submitter_email`, `expense_items`, `period`

---

## Policy limits

Pass `policy_limits` in the request body to override defaults per report, or set organizational defaults directly in the **Policy Check** node's JavaScript constants.

| Limit | Default | Override key |
|---|---|---|
| Meals per diem | $75 | `meal_per_diem` |
| Entertainment cap | $150 | `entertainment_limit` |
| Receipt required above | $25 | `receipt_required_above` |
| Max single item | $500 | `max_single_item` |

For different policies by department (Sales has higher entertainment limits than Admin), pass the appropriate limits in each submission or build a lookup table in the Policy Check node.

---

## Flag severity levels

| Severity | Triggers |
|---|---|
| `high` | Single item over max, entertainment over limit |
| `medium` | Missing receipt above threshold, meal over per diem |
| `low` | Missing date |

**Recommendation logic:**
- Any `high` flags → `review_required`
- More than 2 total flags → `flagged_for_review`
- Only missing receipts → `pending_receipts`
- No flags → `approve`

---

## Expense categories

Free-text but the policy check looks for specific category strings for per-diem and entertainment limits: `meals`, `food`, `entertainment`. Use consistent category names across your submissions. Other categories (travel, accommodation, supplies, parking) pass through without specific limits — add more rules in the Policy Check node as needed.

---

## Line item IDs

Each item gets a `lineId` (L001, L002...) used in flag messages. When a manager sees "missing receipt for L003 ($22.50 taxi)", they know exactly which line to look at. When submitters need to re-submit with fixes, they reference the same IDs.

---

## The manager email

When `manager_email` is provided, the same HTML report goes to the manager with a subject line that includes the recommendation. `review_required` is immediately obvious. The manager clicks through, sees the report with flags highlighted, and can approve or request changes.

---

## Approval workflow

The current workflow handles the intake and review side. It doesn't automate the approval itself — that requires knowing your company's approval authority and finance system.

To add approval: build a second webhook (`/approve-expense`) that accepts a `report_id` and `approved_by`, updates the `status` column in the Reports sheet, and emails the submitter. Add a `/reject-expense` webhook similarly that sends rejection feedback.

---

## Limitations

- Receipt verification is based on the `receipt_provided` boolean in the submission — the workflow trusts the submitter's declaration. Actual receipt file uploads would require a file handling step (Google Drive, S3, etc.) added before the policy check.
- Policy limits are per-item, not cumulative. A week of $74 meals won't trigger the per-diem flag even if total meal spend is high. Add a cumulative check by category in the Policy Check node if needed.
- The workflow doesn't deduplicate submissions — submitting the same report twice creates two entries. Add a check for matching submitter + period + total before saving if deduplication matters.

---

## Ideas

- [ ] Multi-currency support: normalize all items to a base currency using an exchange rate API before totaling
- [ ] OCR receipt processing: accept receipt image uploads, extract amount and merchant via OCR, auto-populate line items
- [ ] ERP integration: push approved reports to QuickBooks, Xero, or SAP via their APIs
- [ ] Category spending dashboard: aggregate the Reports sheet into a monthly spend-by-category chart

---

## License

MIT.
