# FLOOWBOX - Comment Moderation AI Agent

Moderating comments manually across multiple platforms is a full-time job once you start getting traction. This workflow classifies every comment, flags harmful ones instantly, and drafts responses for the ones worth engaging with.

## What it does

Triggered by every new comment via webhook. GPT-4o classifies the comment — question, praise, criticism, spam, or hate — and makes a moderation decision: approve, flag for review, or remove. Drafts a contextual response for approved and flagged comments in the brand voice. Flagged and removed comments trigger a Slack alert for human review. All decisions log to Airtable. Nothing is ever auto-removed — a human confirms removals.

## Tools Used
- **Orchestration:** n8n
- **Classification + Response:** OpenAI GPT-4o
- **Logging:** Airtable
- **Alerts:** Slack
- **Trigger:** Webhook

## Moderation categories

| Decision | Trigger | Action |
|---|---|---|
| Approve | Genuine engagement, questions, criticism | Log + draft response |
| Flag | Potential spam, mildly aggressive | Slack alert + draft response for review |
| Remove | Hate speech, harassment, explicit spam | Slack alert for human confirmation |

## Example output

```json
{
  "decision": "approve",
  "category": "question",
  "should_respond": true,
  "draft_response": "Great question — the short answer is yes, you can connect any webhook-based CRM. We've done it with HubSpot, Zoho, and Airtable. Which CRM are you on? Happy to share the specific setup.",
  "priority": "normal"
}
```

## Connect to your platforms

- **WordPress/website:** Use comment action webhook
- **YouTube:** YouTube Data API comment webhook
- **Instagram:** Instagram Graph API comment webhook
- **Custom:** Any platform that can POST on new comment

## Why I built this

A client with a growing YouTube channel was missing questions in the comments for days — potential customers asking buying questions with no reply. This workflow ensures every actionable comment gets a response drafted within minutes of being posted.

## Setup

1. OpenAI API key
2. Airtable base + Comment Moderation table
3. Slack Bot Token + #moderation channel
4. Connect webhook to your CMS/platform comment system
