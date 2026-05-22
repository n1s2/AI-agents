# FLOOWBOX - Error Log Analyzer

Hourly scan with AI root cause analysis. On-call engineers get a readable digest instead of raw stack traces.

## Tools Used
n8n, Google Sheets, OpenAI GPT-4o, Slack

## Log sheet format
Timestamp | Severity (critical/error/warning) | Message | Component

## Setup
1. Application writes errors to Google Sheets
2. OpenAI API key + Slack Bot + #engineering channel
