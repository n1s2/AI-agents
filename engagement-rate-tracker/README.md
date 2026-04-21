# FLOOWBOX - Engagement Rate Tracker and Analyzer

Posting consistently is only half the job — knowing what actually resonates with your audience is the other half. This workflow calculates weighted engagement rates, identifies what worked, and tells you what to replicate next week.

## What it does

Every Friday at 5 PM, fetches the week's post performance from Google Sheets. Calculates weighted engagement rates — comments and shares count more than likes because they represent deeper engagement. Breaks down performance by platform. GPT-4o analyzes the patterns: what content styles performed above average, what flopped, and specific recommendations for next week's content strategy. Full report to Notion and Slack.

## Tools Used
- **Orchestration:** n8n
- **Data:** Google Sheets (post performance log)
- **Metric Calculation:** Code node (weighted engagement)
- **Pattern Analysis:** OpenAI GPT-4o
- **Storage:** Notion
- **Report:** Slack
- **Schedule:** Weekly Friday 5 PM

## Weighted engagement formula

```
Total Engagements = Likes × 1 + Comments × 3 + Shares × 5 + Saves × 2
Engagement Rate = (Total Engagements / Impressions) × 100
```

Comments and shares are weighted higher because they signal content that made someone stop, think, and act.

## Analysis output

```json
{
  "overall_performance": "good",
  "best_performing_platform": "LinkedIn",
  "what_worked": [
    "Personal story format outperformed tips/lists 3:1 this week",
    "Posts with a contrarian angle got 4x more comments"
  ],
  "content_patterns_to_replicate": [
    "Start with a story from a specific client situation",
    "End with an open question that has no right answer"
  ],
  "content_to_avoid": [
    "Generic tips lists — consistently lowest engagement",
    "Posts without a clear personal angle"
  ]
}
```

## Google Sheets format

| Column | Description |
|---|---|
| Date | Post date |
| Platform | LinkedIn / Twitter / Instagram |
| Topic | Brief description |
| Impressions | Total reach |
| Likes | Like/reaction count |
| Comments | Comment count |
| Shares | Share/repost count |
| Saves | Save/bookmark count |

## Why I built this

FLOOWBOX was posting consistently but not learning from the data — just repeating whatever felt good to write. This weekly analysis revealed that personal client stories consistently outperformed "tips" posts by a wide margin, which completely changed the content approach.

## Setup

1. Google Sheets with post performance data
2. OpenAI API key
3. Notion integration + Analytics DB
4. Slack Bot Token + #content channel
