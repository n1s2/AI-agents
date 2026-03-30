# FLOOWBOX - YouTube Thumbnail A/B Test Tracker

Thumbnail CTR is the single biggest lever for YouTube growth. This workflow tracks performance daily and tells you exactly when to switch thumbnails — before you lose weeks of potential views.

## What it does

Checks YouTube Analytics daily for all videos currently in A/B testing. Pulls view counts, likes, and publish dates via YouTube Data API. GPT-4o analyzes whether the current thumbnail is performing above or below niche averages based on view velocity. When performance is weak and a switch is recommended, an immediate Slack alert fires. All data logs to Airtable for tracking performance over time across thumbnail variants.

## Tools Used
- **Orchestration:** n8n
- **Data:** YouTube Data API v3
- **AI Analysis:** OpenAI GPT-4o
- **Tracking:** Airtable (test log)
- **Alerts:** Slack
- **Schedule:** Daily at 10 AM

## Flow

```
Daily 10 AM
  → Fetch videos with Status = "Testing" (Airtable)
  → YouTube API: get current view/like counts
  → Extract key metrics + days live
  → GPT-4o: analyze performance + switch recommendation
  → Update Airtable with latest data
  → IF switch recommended → Slack alert
```

## What GPT-4o evaluates

```json
{
  "performance_verdict": "weak",
  "switch_thumbnail": true,
  "switch_reason": "Only 340 views in 5 days suggests CTR below 2%",
  "ctr_estimate": "below 2%",
  "winning_elements": ["close-up faces", "high contrast text"],
  "improvement_tips": ["Add emotion to face", "Brighter background"],
  "keep_testing_days": 0
}
```

## Airtable schema

| Field | Type |
|---|---|
| Video ID | Text (YouTube video ID) |
| Current Thumbnail | Select (Version A / B / C) |
| Status | Select (Testing / Complete) |
| Last Check Views | Number |
| Performance | Select |
| AI Recommendation | Long Text |
| Last Checked | Date |

## Why I built this

A YouTube client was manually checking analytics every few days and guessing when to swap thumbnails. They kept thumbnails live too long when performance was weak. This catches underperforming thumbnails within 3-5 days instead of 2-3 weeks.

## Setup

1. YouTube Data API key (Google Cloud Console)
2. Airtable base + Thumbnail Tests table
3. OpenAI API key
4. Slack Bot Token + #youtube channel
