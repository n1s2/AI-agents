# FLOOWBOX - Customer Support Ticket Resolver

Incoming support tickets auto-classified, auto-resolved where possible, escalated with a draft reply where not. Response times go from hours to seconds.

## What it does

Ticket arrives via webhook. GPT-4o classifies by category and priority, drafts a complete email reply, and decides whether it can be auto-resolved. Routine questions get an instant email response. Complex issues escalate to Slack with the draft ready for human review. Everything logs to Airtable.

## Tools Used
- n8n, OpenAI GPT-4o, SMTP, Slack, Airtable

## Setup
1. OpenAI API key
2. SMTP credentials
3. Slack Bot + #support channel
4. Airtable base
