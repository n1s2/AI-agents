# support-escalation-detector

Support tickets that should be escalated often aren't — agents miss subtle signals, miss the context that a customer is high-value, or don't recognize when a complaint pattern is a churn signal. By the time it's obvious, the customer is already gone. This analyzes each ticket thread for escalation risk, churn signals, and root issue, recommends the right action level, and fires an immediate alert for high/critical risk tickets.

---

## What it does

Takes a ticket ID, customer info, account data (tier, MRR, age), and the full message thread. Claude analyzes for:
- Escalation risk (none/low/medium/high/critical)
- Specific signals detected (explicit anger, repeated contacts, manager request, competitor mention, legal threat, etc.)
- Churn risk (none/low/medium/high)
- Churn signals
- Sentiment (calm/frustrated/angry/hostile)
- Root issue (what they actually need, which may differ from what they asked)
- Recommended action: standard_response / prioritize / assign_senior / escalate_to_manager / immediate_call / account_review
- Rationale for the recommendation
- Suggested response opener (first 2 sentences the agent should write — specific, not generic)
- Internal note for support management

For high or critical risk tickets: fires an immediate email alert to the escalation contact.

Returns full analysis as JSON for all tickets regardless of risk level.

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
curl -X POST https://your-n8n.com/webhook/detect-escalation \
  -H "Content-Type: application/json" \
  -d '{
    "ticket_id": "TKT-8841",
    "customer_name": "Rachel Okonkwo",
    "customer_email": "rachel@meridian-ops.com",
    "account_tier": "Enterprise",
    "mrr": 2499,
    "currency": "USD",
    "account_age_days": 847,
    "previous_ticket_count": 2,
    "product_area": "billing",
    "escalation_email": "cs-manager@company.com",
    "assigned_agent": "Tom W.",
    "message_thread": [
      {"role": "customer", "content": "I was charged twice for this month and nobody has responded to my email from 3 days ago. This is unacceptable.", "timestamp": "2025-05-14 09:12"},
      {"role": "agent", "content": "Hi Rachel, I am sorry to hear about this. I have escalated your case internally and will get back to you.", "timestamp": "2025-05-14 11:30"},
      {"role": "customer", "content": "That is what you said last time. I need this resolved today or I am going to have to evaluate whether this platform is worth what we pay. Please have someone senior contact me directly.", "timestamp": "2025-05-14 14:05"}
    ]
  }'
```

**Required:** `ticket_id`, `customer_name`, `message_thread`

---

## Message thread format

Pass as an array of message objects:
```json
{"role": "customer"|"agent", "content": "...", "timestamp": "optional"}
```

Or as a single string for single-message tickets — the validator wraps it automatically.

---

## Escalation signals detected

Claude looks for: explicit anger words, frustrated tone even without explicit words, "I want to cancel" or similar, "speak to a manager" requests, competitor mentions ("we're evaluating X"), legal or compliance language, repeated contacts about the same issue, high-value account with unresolved critical issue, urgency language ("I need this today").

---

## Suggested response opener

The `suggested_response_opener` field gives the agent a specific, non-generic opening they can use or adapt. For the example above, it would acknowledge the double charge specifically and confirm escalation with a name and timeline — not "I'm sorry for the inconvenience."

---

## Limitations

- Analysis is per-submission. The agent doesn't maintain ticket history across separate API calls. For longitudinal context, include previous ticket summaries in the thread or a context field.
- The alert fires for `high` and `critical` risk only. For `medium` risk, the recommendation is returned in the JSON but no email is sent — add a second branch if you want medium-risk alerts too.

---

## Ideas

- [ ] Zendesk/Intercom webhook: trigger this automatically on every new reply in a ticket
- [ ] Slack alert: post high/critical escalations to a dedicated Slack channel instead of (or in addition to) email
- [ ] Priority queue: write escalation risk scores to a Sheet for support lead review dashboard
- [ ] Response drafting: extend the agent to write a full suggested response, not just the opener

---

## License

MIT.
