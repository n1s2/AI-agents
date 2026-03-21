# FLOOWBOX - AI Email Reply Agent

Reads every new email, classifies it, drafts a reply in my writing style, and saves it for review. I check once a day, approve the good drafts, and send. Email time cut from 45 min/day to 5 min.

## What it does

Polls Gmail every 15 minutes for new unread emails. GPT-4o reads each email, classifies it (client inquiry, existing client, partnership, spam), sets a priority level, and writes a reply draft in Navtej's voice. Spam is filtered out automatically. Everything else gets saved to Google Sheets with status "Needs Review" — ready to copy-paste and send.

## Tools Used
- **Orchestration:** n8n
- **Email:** Gmail (OAuth trigger)
- **AI:** OpenAI GPT-4o
- **Storage:** Google Sheets (draft log)
- **Trigger:** Poll every 15 minutes

## Flow
```
Gmail: new unread email arrives
  → Extract sender, subject, body
  → GPT-4o classifies + drafts reply
  → Filter out spam
  → Save to Google Sheets (status: Needs Review)
```

## Email Categories

| Category | Description |
|---|---|
| `client_inquiry` | Someone asking about FLOOWBOX services |
| `existing_client` | Update or question from active client |
| `partnership` | Collaboration or business proposal |
| `spam` | Filtered out, never saved |
| `other` | Everything else |

## Why I built this

Running FLOOWBOX means a lot of inbound email — inquiries, client updates, partnership requests. Reading and replying to each one was eating 45+ minutes every morning. Now GPT-4o handles the first draft and I just review and send. The tone matching in the system prompt means drafts rarely need edits.

## Safety note

This workflow **never sends emails automatically**. Everything goes to a review sheet first. You control what gets sent.

## Setup

1. Connect Gmail via OAuth in n8n credentials
2. Add OpenAI API key
3. Add Google Sheets credentials + create a sheet called "Email Drafts"
4. Update the system message with your own name and company

## Extending this

Add a second workflow that reads approved rows from the sheet and sends them via Gmail. Keep the human-in-the-loop for safety.
