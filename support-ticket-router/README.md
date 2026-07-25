# support-ticket-router

Support ticket routing done manually means a billing question sits in a technical queue for two hours, a critical bug gets triaged as medium severity, and a churning customer gets a canned "thanks for reaching out" response. This reads the actual ticket content, identifies what the customer really needs, classifies severity, detects churn risk, routes to the right team, fires escalation alerts for critical issues, and gives the agent a specific first-response draft.

---

## What it does

Takes ticket subject, body, customer context (name, email, plan, account age, previous ticket count), and your routing teams. Claude produces:

- **Classification** — category (bug/how-to/billing/integration/etc), subcategory, severity (critical/high/medium/low) with rationale
- **Routing** — recommended team, routing rationale, escalate flag, estimated resolution time
- **Ticket analysis** — what the customer actually needs (vs what they asked), churn risk (high/medium/low/none) with specific churn signals, customer sentiment, urgency signals
- **Suggested first response** — specific 3–5 sentence response the agent can adapt (not a generic template)
- **Internal notes** — what the agent should know before responding and what to check first
- **Similar issues to check** — related areas to investigate

For tickets flagged for escalation: fires an immediate plain-text alert to the escalation email.

HTML routing card shows severity badge, routing decision, churn risk, suggested response, and internal notes.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/route-support-ticket \
  -H "Content-Type: application/json" \
  -d '{
    "ticket_id": "TKT-2025-4421",
    "ticket_subject": "Data export not working - critical for client presentation tomorrow",
    "customer_name": "Marcus Webb",
    "customer_email": "marcus@beaconlogistics.com",
    "customer_plan": "Business",
    "account_age_days": 47,
    "previous_ticket_count": 1,
    "product_name": "Flowdesk",
    "escalation_email": "support-lead@flowdesk.com",
    "reply_email": "support@flowdesk.com",
    "routing_teams": [
      {"name": "General Support", "email": "support@flowdesk.com", "specialties": ["how-to", "account", "general"]},
      {"name": "Technical", "email": "tech-support@flowdesk.com", "specialties": ["bugs", "api", "integrations", "performance", "data"]},
      {"name": "Billing", "email": "billing@flowdesk.com", "specialties": ["billing", "payments", "invoices", "plans"]}
    ],
    "ticket_body": "Hi, I have been trying to export our project data to CSV for the last 2 hours and it just spins and never completes. I have 3 projects to export. This is urgent - I need this data for a client presentation tomorrow morning at 9am and I cannot show up without it. I tried logging out and back in. Same issue. Is there a way to do this faster or can someone pull the data for me? This is really stressful."
  }'
```

**Required:** `ticket_subject`, `ticket_body`

---

## Real problem vs stated request

The `real_problem` field often differs from what the customer asked. In the example above, the stated request is "fix the CSV export." The real problem is "I need project data for a presentation tomorrow at 9am." This opens up different resolution paths — maybe an agent can manually pull the export, or email the data directly, even if the underlying bug takes longer to fix.

---

## Churn risk detection

Claude reads for churn signals beyond obvious "I'm cancelling" language — frustrated tone with a new account, multiple unresolved tickets, urgency language around business-critical workflows, phrases like "this is stressful" or "I can't rely on this." High churn risk tickets get flagged so a senior agent or CS manager can prioritize follow-up.

---

## Suggested first response

Not a template — a specific response that references the customer's exact situation. For the example above, it would acknowledge the client presentation deadline specifically, not just "thanks for reaching out." Agents can send it almost as-is or adapt it.

---

## Routing teams

Pass your actual team names, emails, and specialties. Claude matches the ticket to the team whose specialties best fit the classification. You can add as many teams as needed — account management, security, enterprise support, etc.

---

## Limitations

- Classification accuracy depends on ticket content quality. Vague tickets ("it doesn't work") produce lower-confidence routing. The `internal_notes` field flags what information the agent needs to clarify.
- This agent routes one ticket at a time. For high-volume batch routing, call the webhook per ticket from your ticketing platform (Zendesk, Intercom, Freshdesk) via their webhook triggers.

---

## License

MIT.
