# customer-escalation-triager

An escalation that arrives written in all caps isn't necessarily the highest business risk — and a calmly worded email from a champion at a $200k account quietly mentioning they're "evaluating other options" might be far more urgent. This assesses true severity based on business impact and churn risk rather than message tone, identifies specifically who needs to be involved and how fast, and generates a first-response draft. Critical and high-severity escalations trigger an immediate alert to leadership.

---

## What it does

Takes customer name, escalation details, account value, account age, prior escalation count, contract renewal date, CSM assigned, and whether an executive sponsor is already involved. Claude produces:

- **True severity** — critical/high/medium/low, with rationale based on business impact and churn risk, not just tone
- **Churn risk** — high/medium/low with rationale
- **Who needs to be involved** — specific roles with why and urgency (immediate/within_hours/within_day)
- **Response plan** — first response deadline, what the first response should accomplish, resolution approach, follow-up cadence
- **Root issue category** — product_bug/missing_feature/support_quality/communication_breakdown/billing/onboarding_failure/other
- **Suggested first response draft** — ready to send or adapt
- **Internal alert summary** — 2-sentence version suitable for a Slack alert
- **Prevent recurrence** — what would stop this type of escalation from happening again

For critical/high severity: fires an immediate plain-text alert to the escalation email with summary, churn risk, and response deadline.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/triage-escalation \
  -H "Content-Type: application/json" \
  -d '{
    "customer_name": "Meridian Consulting",
    "account_value": 28800,
    "currency": "USD",
    "account_age_months": 14,
    "prior_escalations": 0,
    "contract_renewal_date": "2025-07-30",
    "escalated_by": "Sara (CSM)",
    "customer_contact": "David Chen, VP Operations",
    "contact_seniority": "VP / executive",
    "executive_sponsor_involved": false,
    "csm_assigned": "Sara Kim",
    "escalation_email": "cs-leadership@flowdesk.com",
    "reply_email": "sara@flowdesk.com",
    "escalation_details": "David emailed asking to schedule a call to discuss whether Flowdesk is still the right fit for their team. He mentioned they have had three separate incidents this quarter where task data appeared to sync incorrectly between team members, causing confusion about ownership. He said quote we need to have a serious conversation about reliability before we renew end quote. Tone was calm and professional but direct. Renewal is in 6 weeks."
  }'
```

**Required:** `customer_name`, `escalation_details`

---

## True severity vs message tone

The example above is calmly worded but represents high genuine risk: a senior contact, explicit mention of reliability concerns tied directly to the renewal decision, and a renewal date 6 weeks out. Claude's severity assessment weighs these business factors over the message's emotional register — this would likely be rated `critical` or `high` despite the professional tone, because the churn risk and timing are severe.

---

## Root issue categorization

The `root_issue_category` field helps route the fix appropriately and track patterns over time — a `product_bug` escalation goes to engineering, a `communication_breakdown` might need process fixes in CS, `billing` needs finance involved.

---

## Automatic leadership alert

Escalations assessed as critical or high automatically trigger a plain-text Slack/email alert with the key facts and response deadline — no manual step required to make sure leadership knows about serious risk.

---

## Limitations

- Severity assessment is based on the information provided — richer context (account history, prior sentiment, competitive situation) produces more accurate triage.
- This triages and drafts a response — it doesn't replace human judgment on how to actually navigate the relationship, especially for complex or political situations.

---

## License

MIT.
