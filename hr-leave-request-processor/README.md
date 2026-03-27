# FLOOWBOX - HR Leave Request Processor

Manual leave approval was a daily interruption for managers. This workflow handles 80% of requests automatically — only edge cases reach a human.

## What it does

Employee submits a leave request via an n8n form. The workflow calculates the number of days, checks the employee's leave balance in Airtable, and passes everything to GPT-4o which applies company rules to make a decision: auto-approve, send for manager review, or flag as unusual. Instant approval emails go out for straightforward requests. Complex ones get a Slack message with one-click approve/decline. Everything logs to Airtable.

## Tools Used
- **Orchestration:** n8n
- **Trigger:** n8n Form (no external tool needed)
- **AI:** OpenAI GPT-4o (rule-based decision engine)
- **Data:** Airtable (leave balances + request log)
- **Email:** SMTP (approval notification)
- **Alerts:** Slack (manager review requests)

## Decision Rules

| Condition | Decision |
|---|---|
| Emergency leave (any duration) | Auto-approve |
| Sick leave under 3 days | Auto-approve |
| Annual leave under 5 days + sufficient balance | Auto-approve |
| Annual leave 5+ days | Manager review |
| Insufficient balance | Flagged |

## Flow

```
Employee submits form
  → Calculate days requested
  → Fetch leave balance from Airtable
  → GPT-4o applies rules → decision
  → Auto-approve: send approval email
  → Needs review: Slack to #hr-approvals
  → Log all requests to Airtable
```

## Why I built this

An ops client's managers were getting 15-20 leave request messages per week — mostly 1-2 day sick leaves that obviously should be approved. This auto-handles the simple ones and only surfaces requests that genuinely need judgment.

## Setup

1. Airtable: Leave Balances table + Leave Requests table
2. OpenAI API key
3. SMTP credentials
4. Slack Bot Token + #hr-approvals channel
