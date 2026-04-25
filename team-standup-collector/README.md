# FLOOWBOX - Async Team Standup Collector

No more synchronous standup meetings. Every morning at 9 AM the team gets a prompt, submits async, and by 9:30 a synthesized digest is posted — with blockers flagged and who missed.

## What it does

Posts a standup prompt to Slack at 9 AM. Waits 30 minutes while the team submits responses via Google Form. Fetches today's responses, GPT-4o synthesizes into a team digest, and posts back to Slack — completed work, today's focus, blockers, and who still needs to submit.

## Tools Used
- n8n, OpenAI GPT-4o, Slack, Google Sheets

## Setup
1. Google Form + Sheet for responses (Date, Name, Yesterday, Today, Blockers)
2. OpenAI API key
3. Slack Bot Token + #standup channel
