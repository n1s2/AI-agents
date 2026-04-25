# FLOOWBOX - Expense Report Automator

Monthly expense report generated automatically on the 28th — categorized totals, narrative summary, cost saving suggestions, and a formatted email.

## What it does

Fetches the month's expenses from Google Sheets. Calculates totals by category. GPT-4o writes a plain-English narrative and identifies cost saving opportunities. Sends a formatted HTML email report and logs to monthly history.

## Tools Used
- n8n, Google Sheets, OpenAI GPT-4o, SMTP

## Sheets format
Date, Description, Category, Amount — one row per expense.

## Setup
1. Google Sheets with expense data
2. OpenAI API key, SMTP
