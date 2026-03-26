# FLOOWBOX - Competitor Price Monitor

Knowing what competitors charge — and when they change — is basic business intelligence. I was checking manually every few weeks. Now it runs every morning automatically.

## What it does

Runs daily at 7 AM. Scrapes competitor pricing pages using Jina AI reader. GPT-4o extracts structured plan data — names, prices, billing cycles, features. Saves a daily snapshot to Google Sheets for historical tracking. Aggregates all results and emails a competitive intelligence report with a comparison table and one strategic recommendation.

## Tools Used
- **Orchestration:** n8n
- **Scraping:** Jina AI reader API
- **AI:** OpenAI GPT-4o
- **Storage:** Google Sheets (historical snapshots)
- **Email:** SMTP
- **Schedule:** Daily 7 AM

## Flow
```
7 AM daily
  → Load competitor URLs
  → Scrape each pricing page (Jina AI)
  → GPT-4o extracts plans, prices, free tier, notable changes
  → Save snapshot to Google Sheets
  → Aggregate all competitors
  → GPT-4o writes comparison report
  → Email to founder
```

## Why I built this

A SaaS client discovered a competitor had quietly dropped prices 20% — they found out on a sales call. That's too late. Daily monitoring means you know the same day it happens.

## Setup
1. Add competitor pricing URLs in Set Competitor URLs node
2. Jina AI API key (Header Auth)
3. OpenAI API key
4. Google Sheets ID + SMTP credentials
