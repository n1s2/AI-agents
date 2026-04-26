# FLOOWBOX - Contract Renewal Tracker

Never miss a contract renewal. Daily check flags anything expiring in 7 days (urgent) or 30 days (upcoming), drafts the renewal email, and alerts on Slack — silent on days when nothing needs attention.

## What it does

Fetches all active contracts from Airtable daily. Calculates days until renewal. Flags urgent (≤7 days) and upcoming (≤30 days). GPT-4o recommends specific actions and drafts renewal emails. Posts to Slack only when something needs attention.

## Tools Used
- n8n, Airtable, OpenAI GPT-4o, Slack

## Airtable schema
Contract Name, Party, Renewal Date, Status, Value fields.

## Setup
1. Airtable base with Contracts table
2. OpenAI API key
3. Slack Bot + #contracts channel
