# FLOOWBOX - Product Review Aggregator and Insight Engine

Customer reviews are the most honest product feedback you'll ever get. This workflow collects them weekly and turns them into structured insights.

## What it does

Every Monday at 8 AM, scrapes Amazon reviews for your product and competitor products using Apify. GPT-4o analyzes the full review set and extracts: top praised aspects, most common complaints, frequently requested features, and an overall sentiment score. Saves a weekly insight report to Notion for each product tracked.

## Tools Used
- **Orchestration:** n8n
- **Review Scraping:** Apify (Amazon Reviews Scraper actor)
- **AI Analysis:** OpenAI GPT-4o
- **Storage:** Notion database
- **Schedule:** Weekly, Monday 8 AM

## Flow
```
Monday 8 AM
  → Load product ASINs (yours + competitors)
  → Apify scrapes 50 latest reviews per product
  → Wait 45s for Apify run to complete
  → Fetch results from Apify dataset
  → GPT-4o analyzes and extracts insights
  → Save to Notion page per product
```

## What GPT-4o extracts
```json
{
  "avg_sentiment": "mixed",
  "top_praise": ["fast delivery", "easy setup", "good support"],
  "top_complaints": ["expensive renewal", "mobile app crashes", "missing export"],
  "feature_requests": ["bulk upload", "API access", "Slack integration"],
  "summary": "Customers love the core product but churn over pricing..."
}
```

## Why I built this

A product client was making roadmap decisions based on gut feel. After running this for 4 weeks, they had clear data showing 40% of 1-star reviews mentioned the same missing feature. That became their next sprint.

## Setup
1. Apify account + API key
2. Add product ASINs in Set Products node
3. OpenAI API key
4. Notion integration + Database ID
