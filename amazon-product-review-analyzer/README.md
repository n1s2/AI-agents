# FLOOWBOX - Amazon Product Review Analyzer

Amazon reviews contain everything a seller needs — what buyers love, what breaks, what competitors are doing better, and what would make the product listing convert higher. This workflow extracts all of it automatically.

## What it does

Send a product ASIN. Apify scrapes the 100 most recent reviews. GPT-4o analyzes the full dataset: top praised features, recurring complaints, keywords buyers use (for listing optimization), competitor products mentioned, quality issues to escalate to the product team, listing improvement suggestions, and which negative reviews need a seller response with a draft reply included.

## Tools Used
- **Orchestration:** n8n
- **Scraping:** Apify (Amazon reviews scraper)
- **Analysis:** OpenAI GPT-4o
- **Storage:** Notion
- **Trigger:** Webhook

## Flow

```
POST: {asin, product_name}
  → Apify: scrape 100 recent reviews
  → Wait 45 seconds for scrape to complete
  → Fetch results from Apify dataset
  → Process: avg rating, star distribution, review texts
  → GPT-4o: full sentiment analysis
  → Save to Notion
  → Respond with complete analysis
```

## Analysis output

```json
{
  "overall_sentiment": "mixed",
  "top_praised_features": ["Fast delivery", "Easy setup", "Good build quality"],
  "top_complaints": ["Battery life", "Customer support response time"],
  "listing_improvement_suggestions": [
    "Add battery life specs to bullet points — 23 reviews mention it",
    "Include setup video link — 'easy to set up' is top praise but buyers mention PDF confusing"
  ],
  "response_needed_reviews": [
    {
      "issue": "Defective unit received",
      "review_type": "1-star",
      "suggested_response": "We're sorry to hear about the defective unit. Please contact support@... and we'll replace it immediately."
    }
  ],
  "product_development_insights": ["Battery life is the #1 improvement request — mentioned in 34% of reviews"]
}
```

## Why I built this

An Amazon seller client was manually reading reviews to write listing updates. After analyzing 200+ reviews per product manually, they were still missing patterns visible only in aggregate. This workflow surfaces those patterns in minutes.

## Setup

1. Apify account + API key
2. OpenAI API key
3. Notion integration + DB ID
