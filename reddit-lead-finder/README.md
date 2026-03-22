# FLOOWBOX - Reddit Lead Finder for B2B Services

Runs every 6 hours, scans Reddit for people asking for exactly what FLOOWBOX offers, scores each post by lead quality, and saves the best ones with a reply draft ready to go.

## What it does

Searches Reddit for posts mentioning automation needs across subreddits like r/entrepreneur, r/smallbusiness, r/startups. GPT-4o-mini scores each post 1-10 for lead quality and writes a ready-to-post helpful reply. Only posts scoring 7+ get saved to Google Sheets. Runs automatically every 6 hours.

## Tools Used
- **Orchestration:** n8n
- **Data:** Reddit API (no auth needed for search)
- **AI:** OpenAI GPT-4o-mini (scoring + reply generation)
- **Storage:** Google Sheets
- **Trigger:** Schedule (every 6 hours)

## Flow
```
Every 6 hours
  → Search Reddit (25 newest posts matching query)
  → Split into individual posts
  → Extract title, text, URL, author, subreddit
  → GPT-4o-mini: score lead quality + draft reply
  → Filter: keep only score 7+
  → Save to Google Sheets with status "New"
```

## Lead Scoring Logic

GPT-4o-mini scores strictly:
- **8-10:** Explicitly looking for automation help, sounds ready to hire
- **5-7:** Related problem but not clearly looking for paid help
- **1-4:** Venting, general discussion, not a lead

Only 7+ makes it to your sheet.

## Why I built this

Finding leads on Reddit manually was hit or miss. This runs in the background and surfaces only the high-intent posts — I check the sheet once a day and reply to 2-3 genuinely good leads. No spam, no irrelevant posts.

## Setup

1. No Reddit API key needed — uses public JSON endpoint
2. Add OpenAI API key
3. Add Google Sheets credentials + your Sheet ID
4. Customize `search_query` and `my_service` to match what you offer

## Customization

Change `subreddits` to target your specific niche. Change `search_query` to match your service keywords. The more specific, the better the lead quality.
