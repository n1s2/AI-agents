# FLOOWBOX - WhatsApp Lead Capture and CRM Sync

One of the most common problems I see with small business clients — they get 20-30 WhatsApp inquiries daily and manually copy each one into a spreadsheet or CRM. This workflow eliminates that completely.

## What it does

Every incoming WhatsApp message hits a webhook. GPT-4o reads the message and classifies the intent — is this a buying signal, a general inquiry, or spam? Based on that, the lead gets routed: hot leads go straight into Airtable as high-priority, inquiries get a friendly auto-reply, spam gets ignored. The whole thing takes under 3 seconds.

## Tools Used
- **Orchestration:** n8n
- **AI:** OpenAI GPT-4o (intent classification)
- **Messaging:** WhatsApp Business API (Meta)
- **CRM:** Airtable
- **Logic:** Webhook trigger, switch routing, auto-reply

## Flow

```
WhatsApp Message Received (Webhook)
  → Extract: name, phone, message, timestamp
  → GPT-4o classifies intent + urgency
  → Route by intent:
      Buying  → Add to Airtable as Hot Lead + Auto Reply
      Inquiry → Auto Reply only
      Spam    → Ignore + Webhook 200 response
  → Webhook response sent
```

## Why I built this

A retail client was losing leads because they couldn't respond fast enough — sometimes 4-5 hours delay on WhatsApp. This workflow responds instantly and makes sure nothing falls through the cracks. Hot leads are flagged in CRM within seconds of the first message.

## Setup

1. Meta Developer account → WhatsApp Business API → get Phone Number ID + Token
2. Configure Webhook URL in Meta dashboard → point to n8n webhook
3. Add your Airtable Base ID and table name
4. Set OpenAI credentials

## What the AI extracts

```json
{
  "intent": "buying",
  "product_interest": "automation services",
  "urgency": "high",
  "recommended_action": "Call within 1 hour"
}
```
