# customer-support-triage

The gap between a ticket arriving and a useful response going out is where customer satisfaction gets made or lost. Most support queues treat a billing question the same as a critical bug, route everything to the same team, and send the same acknowledgement email regardless of the customer's tone or situation.

This takes an incoming support ticket, classifies it across eight dimensions (category, urgency, sentiment, routing, escalation, SLA, churn risk, internal note), drafts a complete first response the agent can send immediately with minimal editing, logs to Google Sheets, and posts to Slack for the right team. The auto-response goes out in under 30 seconds.

The key difference from basic auto-responders: Claude writes a specific draft, not a template. A frustrated customer who can't log in gets a different response than a curious customer asking about pricing.

---

## What it does

1. Accepts a POST: customer message, email, name, plan, account value, product, channel, previous ticket count, optional knowledge base snippets
2. Claude triages across:
   - Issue category (billing, technical bug, feature request, how-to, account access, complaint, cancellation, data question, integration, other)
   - Urgency (critical / high / medium / low) with rationale
   - Sentiment (very negative / negative / neutral / positive)
   - Routing (tier1 / tier2 / billing / account management / engineering / product / legal)
   - Escalation needed flag with reason
   - SLA recommendation (specific time window)
   - Draft response: complete email ready to send, references KB if provided
   - Internal note for the agent
   - Churn risk (high / medium / low / none) with rationale
3. Logs to Google Sheets
4. Sends auto-response email to customer
5. If `team_slack_channel` provided: posts triage summary to Slack
6. Returns full triage JSON

---

## Stack

- **n8n** — webhook + workflow
- **Google Sheets** — ticket log
- **Anthropic Claude** (claude-sonnet-4-20250514) — triage + response writing
- **SMTP** — auto-response to customer
- **Slack** — team notification

---

## Setup

### 1. Create the Tickets sheet

One tab: **Tickets** — columns:
```
ticket_id | submitted_at | customer_email | customer_name | customer_plan | issue_category | urgency | sentiment | routing | escalation_needed | churn_risk | status
```

### 2. Environment variables

```
SUPPORT_SHEET_ID=your_google_sheet_id
SUPPORT_EMAIL=support@yourcompany.com
```

### 3. Credentials

- **Google Sheets OAuth2**
- **Anthropic API** (LangChain node)
- **SMTP**
- **Slack API** (optional)

### 4. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/support-ticket \
  -H "Content-Type: application/json" \
  -d '{
    "customer_message": "Hi, I was charged twice for my subscription this month. I see two identical charges of $79 on my statement from May 3rd and May 5th. This has never happened before and I need this resolved immediately. I have been a customer for 3 years.",
    "customer_email": "fatima@businesscorp.com",
    "customer_name": "Fatima Al-Hassan",
    "customer_id": "cust_8847",
    "customer_plan": "Business",
    "account_value": 948,
    "currency": "USD",
    "channel": "email",
    "product": "Acme SaaS",
    "previous_ticket_count": 1,
    "team_slack_channel": "#support-billing",
    "knowledge_base_snippets": [
      "Duplicate charge policy: If a customer reports a duplicate charge, the finance team can issue a refund within 1 business day. Refunds appear within 3-5 business days.",
      "Escalation threshold: Any billing issue over $100 from a customer over 12 months tenure should be routed to account management."
    ]
  }'
```

**Required:** `customer_message`, `customer_email`

---

## The draft response

The `draft_response` field is a complete email — not a template with placeholders. For the example above, it would say something like:

> "Hi Fatima, thank you for reaching out and I'm sorry about this — a double charge is frustrating, especially after three years with us. I can see the two $79 charges on May 3rd and May 5th and I'm initiating a refund for the duplicate right now. You'll see it back on your statement within 3–5 business days. I've also flagged this with our billing team to make sure it doesn't happen again. If you have any other questions, I'm here."

Agents can send this as-is or make minor adjustments. No blank fields to fill.

---

## Knowledge base snippets

Pass up to 5 knowledge base snippets as strings in `knowledge_base_snippets`. Claude incorporates relevant ones into the draft response. This is the fastest way to inject product-specific knowledge without building a RAG pipeline.

Effective snippets:
- Policy statements ("Refunds for annual plans are prorated within 30 days of renewal")
- Troubleshooting steps for common issues
- Contact info for specific teams ("Enterprise issues: escalate to success@company.com")
- Pricing or plan details

---

## Urgency classification

| Level | Typical triggers |
|---|---|
| `critical` | Service outage, data loss, security issue, cancellation intent |
| `high` | Billing error, can't access account, significant functionality broken |
| `medium` | Feature question, moderate bug, general complaint |
| `low` | How-to question, feature request, general inquiry |

High-value accounts (`account_value` above a threshold) can upgrade the urgency — a medium issue from a $10k/year account gets treated like a high one. Claude factors this in.

---

## Churn risk

Claude assesses churn risk based on:
- Explicit cancellation mention
- Sentiment combined with severity ("I've had this issue three times")
- Account tenure and ticket history (`previous_ticket_count`)
- Account value

High churn risk triggers an explicit flag in the Slack notification.

---

## Routing options

`tier1_support` — standard first-line issues
`tier2_technical` — bugs, integrations, technical deep-dives
`billing_team` — payment, subscription, invoice issues
`account_management` — high-value customers, renewals, expansions
`engineering` — confirmed bugs, data issues
`product_team` — feature requests, product feedback
`legal` — compliance, data deletion, disputes

---

## Auto-response vs human review

The draft response fires automatically as an auto-reply with the ticket ID in the subject. This sets expectations and often resolves simple issues without human intervention.

For workflows where you want human review before sending:
- Set `send_auto_response: false` in the webhook body (add a conditional check in the workflow)
- The draft still appears in the JSON response and Slack notification for the agent to review and send manually

---

## Limitations

- Claude drafts based on the customer message and KB snippets alone. For issues requiring account lookup (actual charge dates, usage data, account history), the agent needs to verify before sending.
- The auto-response goes out immediately. If your team wants to review before first contact, add a 30-minute delay or make auto-send conditional on urgency level.
- Churn risk and sentiment are assessments of the message, not the customer relationship. They're useful signals, not definitive scores.

---

## Ideas

- [ ] Zendesk/Intercom integration: POST incoming tickets directly from your helpdesk, push the triage results back as tags and assignments
- [ ] Resolution tracking: a companion `/resolve-ticket` webhook to close tickets and track resolution time
- [ ] CSAT follow-up: 24 hours after a ticket is resolved, auto-send a satisfaction survey
- [ ] Pattern detection: weekly digest of ticket categories and churn risk flags for product and CS leadership

---

## License

MIT.
