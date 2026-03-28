# FLOOWBOX - NPS Survey Auto-Analyzer

Most NPS tools show you a score. This workflow reads the actual comments, understands what customers mean, and takes action automatically — within seconds of each response.

## What it does

Every NPS response hits the webhook instantly. The workflow classifies the respondent (promoter / passive / detractor) and sends the comment to GPT-4o for deep analysis — extracting sentiment topics, product feedback, churn risk, expansion opportunities, and feature requests. Detractors trigger an urgent Slack alert with recommended action. Promoters get a personalized thank-you email with a referral ask. Everything logs to Google Sheets for trend analysis.

## Tools Used
- **Orchestration:** n8n
- **AI:** OpenAI GPT-4o (comment analysis)
- **Alerts:** Slack (#nps-urgent for detractors)
- **Email:** SMTP (promoter follow-up)
- **Logging:** Google Sheets
- **Trigger:** Webhook (connect to Typeform, Surveymonkey, or custom form)

## NPS Categories

| Score | Category | Action |
|---|---|---|
| 9-10 | Promoter 🟢 | Thank you email + referral ask |
| 7-8 | Passive 🟡 | Log + add to nurture campaign |
| 0-6 | Detractor 🔴 | Urgent Slack alert + churn flag |

## What GPT-4o extracts per response

```json
{
  "sentiment_topics": ["slow onboarding", "loves the automation features"],
  "product_feedback": "Wishes the dashboard had better filtering",
  "churn_risk": true,
  "expansion_opportunity": false,
  "suggested_response": "Hi Priya, I'm sorry to hear onboarding felt slow...",
  "internal_action": "Schedule call with CS team this week",
  "feature_request": "Advanced dashboard filtering"
}
```

## Why I built this

A client was collecting NPS responses in a spreadsheet and reviewing them monthly — by which time detractors had already churned. This acts on every response in real time. Detractor response time went from weeks to minutes.

## Setup

1. Connect your NPS tool to POST responses to webhook URL
2. OpenAI API key
3. Slack Bot Token + #nps-urgent channel
4. SMTP credentials
5. Google Sheets ID for logging
