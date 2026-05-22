# FLOOWBOX - Database Backup Notifier

Daily check that every database has been backed up within its configured window. Stale → alert. All healthy → quiet digest.

## Tools Used
n8n, Google Sheets, Slack

## Sheets format
Database | Last Backup (ISO date) | Max Age Hours

## Setup
1. Backup system writes last backup time to Google Sheets after each run
2. Slack Bot + #alerts + #devops channels
