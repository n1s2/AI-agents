# FLOOWBOX - Brand Mention Response Agent

Every unanswered complaint is a missed chance to turn a critic into a customer. Every unanswered question is a missed sale. This workflow scans Twitter and Reddit hourly for brand mentions and drafts responses in your brand voice — ready to post in one click.

## What it does

Runs every hour. Searches Twitter API for recent mentions and Reddit for posts containing brand keywords. GPT-4o classifies each mention — question, complaint, praise, or neutral discussion — assigns priority, and drafts a response for anything worth replying to. The drafts appear in Slack for human review. Nothing posts automatically — a human approves every response before it goes live.

## Tools Used
- **Orchestration:** n8n
- **Twitter:** Twitter API v2
- **Reddit:** Reddit public JSON API
- **Classification + Drafting:** OpenAI GPT-4o
- **Review Queue:** Slack
- **Schedule:** Hourly

## Classification output

```json
{
  "classified": [
    {
      "platform": "Reddit",
      "type": "question",
      "priority": "high",
      "text_snippet": "Anyone tried FLOOWBOX for automating client reporting?",
      "respond": true,
      "draft_response": "Hey! Yes — we have a client reporting workflow that pulls from Sheets and generates a Slack digest automatically. Happy to share the setup if useful. What's your current reporting process?",
      "why": "Direct question from potential customer with high intent"
    }
  ]
}
```

## Why this beats manual monitoring

Checking Twitter and Reddit manually means checking at best 2-3 times per day. Mentions from 6+ hours ago get no response — the person has moved on. Hourly monitoring means responding within 1-2 hours, when the conversation is still warm.

## Setup

1. Twitter Developer account + API Bearer Token (OAuth2)
2. OpenAI API key
3. Slack Bot Token + #brand-mentions channel
4. Update brand keywords in Set Brand Config
