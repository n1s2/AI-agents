# FLOOWBOX - Vendor Onboarding Automator

New vendor approved — this workflow creates their Airtable record, sends a personalized welcome email with the document checklist, and notifies the internal team with their action items. Done in under 30 seconds.

## What it does

Webhook fires when a new vendor is approved. GPT-4o generates a service-specific onboarding plan — required documents, setup steps, systems to provision, and a go-live estimate. Creates a vendor record in Airtable, sends a welcome email, and posts internal tasks to Slack.

## Tools Used
- n8n, OpenAI GPT-4o, Airtable, SMTP, Slack

## Setup
1. Airtable base with Vendors table
2. OpenAI API key, SMTP credentials
3. Slack Bot + #operations channel
