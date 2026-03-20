# FLOOWBOX - Trustpilot Scraper + AI Sentiment Analyzer

Brand reputation monitoring, automated. Built for a client who needed to track competitor sentiment shifts without checking manually.

## What it does

Scrapes Trustpilot reviews for any company using their slug. DeepSeek extracts structured data from each review (rating, topic, sentiment indicators). OpenAI then runs a sentiment analysis pass across the full review set to identify patterns and flag significant shifts.

## Tools Used
- **Orchestration:** n8n
- **Data Extraction:** DeepSeek
- **Sentiment Analysis:** OpenAI
- **Logic:** Conditional branching, pagination, limit controls

## Config
Set `company_name` to the Trustpilot URL slug of the company you want to monitor. Adjust `max_pages` based on review volume needed.

## Why I built this
A client wanted weekly competitor reputation monitoring. Manual Trustpilot checking took hours. This runs on a schedule and flags anything worth reviewing.
