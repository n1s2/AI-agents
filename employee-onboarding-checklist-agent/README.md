# FLOOWBOX - Employee Onboarding Checklist Agent

Every new hire used to mean 3 hours of manual setup — creating accounts, writing welcome emails, building checklists. This workflow does all of it the moment HR submits a form.

## What it does

Webhook fires when a new employee is added. GPT-4o generates a role-specific 30-day onboarding plan — week by week tasks, tools to set up, people to meet, success metrics at day 30. Creates a Notion page with the full plan, sends a personalized welcome email to the employee with their Week 1 schedule, and notifies the manager on Slack.

## Tools Used
- **Orchestration:** n8n
- **AI:** OpenAI GPT-4o (role-specific plan generation)
- **Storage:** Notion (onboarding database)
- **Email:** SMTP (employee welcome)
- **Alerts:** Slack (manager notification)
- **Trigger:** Webhook (connect to HRIS or Google Form)

## Flow

```
HR submits new hire form (Webhook)
  → Extract: name, email, role, department, start date
  → GPT-4o generates 30-day onboarding plan
  → Create Notion page with full plan
  → Send welcome email to employee (Week 1 preview)
  → Slack alert to manager
```

## What GPT-4o generates

```json
{
  "week_1": [{"day": 1, "tasks": ["Meet team", "Set up laptop", "Read company handbook"], "owner": "Manager"}],
  "week_2": [{"focus": "Product deep-dive", "tasks": ["Shadow customer calls", "Review codebase"]}],
  "week_3_4": [{"focus": "First deliverable", "tasks": ["Complete assigned project", "1:1 with manager"]}],
  "tools_to_setup": ["Slack", "Notion", "GitHub", "Linear"],
  "people_to_meet": ["CTO", "Head of Design", "3 engineers"],
  "success_metrics": ["Shipped first PR", "Completed onboarding doc", "Led one team meeting"]
}
```

## Why I built this

A client hiring 20+ people per year was spending 3 hours per hire on manual onboarding setup. The personalization by role was the key — a frontend engineer and a sales hire need completely different week 1 experiences. GPT-4o handles that variation automatically.

## Setup

1. Connect HR system or Google Form to webhook
2. OpenAI API key
3. Notion integration + Database ID
4. SMTP credentials
5. Slack Bot Token
