# contract-renewal-tracker

Vendor contracts with auto-renewal clauses are a quiet source of wasted spend. Notice periods pass unnoticed, contracts renew automatically at full price, and nobody remembers to evaluate whether the tool is even being used. This logs contracts to a registry and runs a daily check that surfaces what needs action — specifically calling out contracts where the cancellation notice deadline is approaching or has already passed.

---

## What it does

**Log contract (webhook `/log-contract`):** Saves a contract to the registry with renewal date, value, auto-renew flag, notice period, owner, and satisfaction notes.

**Daily check (8am):** Loads all active contracts, identifies those renewing in the next 120 days, calculates the notice deadline for each (renewal date minus notice period days), and flags urgent ones (notice deadline within 14 days) and auto-renew risks (notice deadline already passed on an auto-renewing contract). Claude reviews satisfaction notes and gives a renew/renegotiate/cancel/investigate recommendation per urgent contract, then sends a digest email.

---

## Stack

n8n (webhook + daily scheduler), Google Sheets, Anthropic Claude (claude-sonnet-4-20250514), SMTP.

---

## Setup

**Sheet "Contracts"** columns: `contract_id | contract_name | vendor_name | vendor_contact | renewal_date | annual_value | currency | auto_renews | notice_period_days | contract_owner | owner_email | category | satisfaction_level | usage_notes | status`

**Env vars:** `CONTRACT_SHEET_ID`, `FROM_EMAIL`, `CONTRACT_OWNER_DEFAULT_EMAIL`

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/log-contract \
  -H "Content-Type: application/json" \
  -d '{
    "contract_name": "Salesforce Enterprise",
    "vendor_name": "Salesforce",
    "renewal_date": "2025-08-15",
    "annual_value": 84000,
    "currency": "USD",
    "auto_renews": true,
    "notice_period_days": 60,
    "contract_owner": "Sarah Chen",
    "owner_email": "sarah@company.com",
    "category": "CRM",
    "satisfaction_level": "moderate",
    "usage_notes": "Only using 40% of licensed seats. Considering downsizing at renewal."
  }'
```

**Required:** `contract_name`, `vendor_name`, `renewal_date`, `annual_value`, `currency`

---

## Notice deadline logic

For an auto-renewing contract with a 60-day notice period renewing August 15, the notice deadline is June 16 — the last day you can cancel without the auto-renewal kicking in. The agent flags this as urgent when within 14 days of that deadline, and as an "auto-renew risk" if that deadline has already passed.

---

## Limitations

- This is a tracking and alerting tool, not an e-signature or contract management system. It doesn't store the actual contract document.
- Recommendations are based on whatever satisfaction/usage notes you log — garbage in, garbage out. Update these notes periodically for accurate recommendations.

---

## License

MIT.
