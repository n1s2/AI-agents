# FLOOWBOX - Customer Feedback Loop Agent

Feedback submitted → customer gets a personal thank-you instantly, the right team gets an internal note, and high-priority items hit Slack before the hour is out. No feedback falls into a void.

## What it does

Feedback arrives via webhook. GPT-4o classifies the type (bug, feature request, praise), assigns priority, identifies which team should own it, and extracts a product insight. Sends a personalized thank-you email. Logs to Airtable. Fires a Slack alert for high priority items.

## Tools Used
- n8n, OpenAI GPT-4o, SMTP, Airtable, Slack

## Setup
1. OpenAI API key, SMTP credentials
2. Airtable base + Customer Feedback table
3. Slack Bot + #product channel
