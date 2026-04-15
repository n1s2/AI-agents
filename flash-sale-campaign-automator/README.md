# FLOOWBOX - Flash Sale Campaign Automator

Launching a flash sale used to mean 2 hours of writing emails and social posts before you could even start. This workflow launches a complete multi-channel sale campaign in under 2 minutes — email to entire list, all social copy ready, and an automated final urgency email before the sale ends.

## What it does

Trigger via webhook with sale details. GPT-4o simultaneously writes the email campaign (subject, preview, body) and all social media copy (Instagram, Twitter, LinkedIn, WhatsApp). Sends the email to the full customer list immediately. Posts all social copy to Slack for one-click posting. Then waits and automatically sends a final urgency email 2 hours before the sale expires.

## Tools Used
- **Orchestration:** n8n
- **Copy Writing:** OpenAI GPT-4o (parallel — email + social)
- **Email:** SMTP
- **Customer List:** Google Sheets
- **Social Copy Delivery:** Slack
- **Trigger:** Webhook

## Flow

```
POST: {sale_name, discount, products, hours, code, segment}
  → Parallel:
      GPT-4o: write email campaign
      GPT-4o: write Instagram/Twitter/LinkedIn/WhatsApp
  → Send emails to customer list
  → Post social copy to Slack
  → Wait (sale_duration - 2 hours)
  → GPT-4o: write final urgency email
  → Send final email blast
```

## Webhook payload

```json
{
  "sale_name": "Weekend Flash Sale",
  "discount": 20,
  "products": ["Automation Starter Kit", "Pro Bundle"],
  "hours": 24,
  "code": "FLASH20",
  "segment": "all customers"
}
```

## What GPT-4o generates

**Email subject:** `⚡ 20% off ends in 24hrs (no code needed)`

**Instagram:** `🔥 FLASH SALE — 20% off everything for the next 24 hours only...`

**WhatsApp:** `Hey! Quick one — we're running a 24hr flash sale. 20% off with code FLASH20. Ends tomorrow 9 AM. Shop here: [link]`

## Why I built this

A client was manually writing all sale copy every time, then scheduling it individually. The whole setup took 2-3 hours, which often delayed launching until the "good window" had passed. This compresses the entire launch to under 5 minutes.

## Setup

1. Google Sheets with customer emails
2. OpenAI API key
3. SMTP credentials
4. Slack Bot Token + #marketing channel
