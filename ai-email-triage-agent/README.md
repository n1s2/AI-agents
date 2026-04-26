# FLOOWBOX - AI Email Triage Agent

Every incoming email classified, prioritized, and routed — automatically. Urgent items get an instant Slack alert with a draft reply. Everything else gets logged to a Notion inbox tracker.

## What it does

IMAP trigger fires on every new email. GPT-4o classifies the sender type, priority, required action, and deadline. Drafts a reply if one is needed. Urgent emails go to Slack immediately. All emails log to Notion with structured metadata for easy filtering.

## Tools Used
- n8n (IMAP trigger), OpenAI GPT-4o, Slack, Notion

## Setup
1. IMAP email credentials
2. OpenAI API key
3. Slack Bot + #inbox-urgent channel
4. Notion integration + Inbox DB
