# customer-churn-survey-analyzer

Exit surveys collect a stated reason for cancellation that's often not the real reason. "Too expensive" frequently means "didn't see enough value to justify the price" — a different problem with a different fix. This analyzes each churn response individually to identify the underlying driver, assess save potential, and flag urgent cases — then rolls everything into a weekly pattern digest for product and CS leadership.

---

## What it does

**Per-response (webhook):**
- Accepts a churn survey submission: stated reason, free-text feedback, account details, NPS, competitor switched to
- Claude categorizes into one of 10 standard categories, identifies the underlying reason (which may differ from what was stated), assesses sentiment, rates save potential, flags urgent cases
- Logs to Google Sheets
- Sends an immediate alert to CS lead for urgent flags (high-value account, fixable issue, recoverable situation)

**Weekly digest (Monday 8am):**
- Aggregates the past week's responses
- Claude identifies the dominant pattern and whether it differs from typical churn drivers
- Reports total MRR lost and count of saveable accounts
- Emails formatted digest with category breakdown

---

## Stack

n8n, Google Sheets, Anthropic Claude (claude-sonnet-4-20250514), SMTP.

---

## Setup

**Sheet "Responses"** columns: `response_id | submitted_at | customer_name | account_tier | mrr | currency | stated_reason | primary_category | underlying_reason | sentiment | save_potential | urgent_flag | winback_interest | would_recommend`

**Env vars:** `CHURN_SURVEY_SHEET_ID`, `FROM_EMAIL`, `CS_LEAD_EMAIL`

**Credentials:** Google Sheets OAuth2, Anthropic API, SMTP

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/churn-survey-response \
  -H "Content-Type: application/json" \
  -d '{
    "customer_name": "Beacon Logistics",
    "customer_email": "ops@beaconlog.com",
    "account_tier": "Pro",
    "months_as_customer": 14,
    "mrr": 299,
    "currency": "USD",
    "cancellation_reason": "Too expensive",
    "free_text_feedback": "We like the product but honestly we only use about 30% of the features and our new ops lead questioned why we are paying this much. We never got a proper onboarding walkthrough of the advanced reporting which is probably the part that would have justified the cost.",
    "would_recommend": 6,
    "winback_interest": true
  }'
```

**Required:** `customer_name`, `cancellation_reason`, `free_text_feedback`

For this example, Claude would likely identify the underlying reason as poor onboarding/feature adoption rather than price, and flag medium-high save potential since the customer is open to winback.

---

## Categories

`price`, `missing_feature`, `poor_support`, `found_alternative`, `no_longer_needed`, `poor_onboarding`, `bug_reliability`, `usability`, `business_closed`, `other`

---

## Limitations

Analysis quality depends on free-text feedback richness — sparse feedback produces sparse insight. This doesn't automate winback outreach; `winback_interest` is captured but acting on it is manual or requires a separate workflow.

---

## License

MIT.
