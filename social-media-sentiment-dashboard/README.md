# FLOOWBOX - Social Media Sentiment Dashboard

Knowing what people say about your brand across Twitter, Reddit, and news — every single day — without manually checking each platform. This workflow aggregates everything into one daily sentiment score.

## What it does

Every morning at 7 AM, simultaneously searches Twitter API for recent mentions, Reddit for keyword discussions, and Perplexity for news coverage. GPT-4o analyzes all three sources together and generates a unified sentiment score, highlights top positive and negative mentions, surfaces trending topics, flags competitor mentions, and writes a 2-3 sentence daily summary. Results log to Google Sheets for trend tracking and a digest posts to Slack.

## Tools Used
- **Orchestration:** n8n
- **Twitter:** Twitter API v2
- **Reddit:** Reddit public JSON API
- **News:** Perplexity AI Sonar
- **Analysis:** OpenAI GPT-4o
- **Logging:** Google Sheets (daily trend)
- **Report:** Slack
- **Schedule:** Daily 7 AM

## Flow

```
7 AM daily
  → Parallel fetch:
      Twitter API (last 50 mentions)
      Reddit API (last 25 posts, last 24h)
      Perplexity (news mentions last 24h)
  → Aggregate all sources
  → GPT-4o unified sentiment analysis
  → Log to Google Sheets
  → Slack daily digest
```

## Daily report output

```json
{
  "overall_sentiment": "positive",
  "sentiment_score": 7.2,
  "total_mentions": 23,
  "trending_topics": ["n8n v1.8 release", "AI automation ROI"],
  "top_positive_mentions": [{"platform": "Twitter", "text": "FLOOWBOX saved us 10 hours/week", "reach": 1200}],
  "action_items": ["Respond to Reddit thread in r/entrepreneur"],
  "daily_summary": "Sentiment positive today driven by Twitter discussion of automation ROI. One Reddit question needs a response."
}
```

## Why I built this

Reputation management for a client required checking 4 platforms daily. Missing a negative mention for 48 hours let a complaint gain traction. This surfaces everything within hours of it being posted.

## Setup

1. Twitter Developer account + API Bearer Token (OAuth2)
2. Perplexity API key
3. OpenAI API key
4. Google Sheets ID
5. Slack Bot Token + #brand-monitoring channel
