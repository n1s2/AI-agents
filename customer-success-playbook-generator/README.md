# customer-success-playbook-generator

CS playbooks that consist of "check in with the customer regularly" aren't playbooks. They're aspirations. A real playbook tells CSMs exactly what to do, when, through which channel, with what words — and what to look for if things go off track. This generates a complete, prescriptive playbook for any CS motion with phase-by-phase actions, email templates, talk tracks, and escalation triggers.

---

## What it does

Takes playbook type, product name, customer segment, activation milestones, success metrics, and churn signals. Claude generates:

- Playbook title, summary, goal, and trigger
- **Phases** — each with timeframe, objective, CSM actions (with channel, timing, and actual scripts/templates), customer goals for the phase, and a risk check
- **Escalation triggers** — specific signals with what to do and urgency level
- **Email templates** — complete email bodies with subject lines, ready to send or adapt
- **Talk tracks** — scenario, opening line, key points to cover, anticipated objection, and response
- **Success definition** — measurable metrics and timeline

HTML output with phase cards, template blocks with the actual email copy, and talk track objection/response boxes.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-cs-playbook \
  -H "Content-Type: application/json" \
  -d '{
    "playbook_type": "onboarding",
    "product_name": "Flowdesk",
    "company_description": "Lightweight project management for ops teams at small companies",
    "customer_segment": "mid_market",
    "avg_contract_value": 12000,
    "currency": "USD",
    "cs_team_size": 3,
    "tool_stack": "Salesforce, Intercom, Zoom, Google Docs",
    "activation_milestones": [
      "First project created",
      "3 team members added",
      "First task assigned and completed",
      "Slack integration connected",
      "First weekly review completed"
    ],
    "key_success_metrics": [
      "Time to first project: under 48 hours from signup",
      "Team activation: 80% of licensed users active within 30 days",
      "Feature adoption: Slack integration connected within 14 days"
    ],
    "typical_churn_signals": "Login frequency drops below 2x/week for admin user. Team members not being added in first 2 weeks. No tasks completed after day 7. CS call declined 2+ times in a row.",
    "reply_email": "cs-lead@flowdesk.com"
  }'
```

**Required:** `playbook_type`, `product_name`

---

## Playbook types

`onboarding`, `qbr`, `renewal`, `expansion`, `at_risk_recovery`, `executive_sponsor`, `churn_prevention`

Each type gets a fundamentally different structure. Onboarding is phase-by-phase from signup to activation. QBR is structured around business review agenda. At-risk recovery is trigger-based with escalating interventions. Renewal focuses on value articulation and contract negotiation.

---

## Email templates

Every generated template is a complete, ready-to-send email — subject line and full body. Not a fill-in-the-blank template with [CUSTOMER NAME] everywhere, but a real draft that the CSM can adapt in 30 seconds. The template references the product, the milestone, and the value proposition specific to that phase.

---

## Talk tracks

Each talk track includes the specific scenario (when to use it), the first line to say, key points to cover, the most common objection the customer raises at this stage, and how to respond to it. These aren't scripts — they're guides for what to say when the conversation goes in a direction the CSM hasn't prepared for.

---

## Limitations

- Playbooks are starting points — CSMs should adapt based on actual customer behavior and their specific product. Review email templates for product accuracy before using.
- The agent generates one playbook at a time for one customer segment. For multiple segments (SMB vs enterprise) run separately with appropriate segment parameters.

---

## License

MIT.
