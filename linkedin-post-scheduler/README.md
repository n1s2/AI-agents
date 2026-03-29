# FLOOWBOX - LinkedIn Post Scheduler and Repurposer

LinkedIn requires a specific writing style — short paragraphs, personal angles, question endings. This workflow takes approved content and automatically formats it for LinkedIn, then schedules it for the optimal posting time.

## What it does

Every Sunday evening, fetches up to 5 approved content items from a Notion database. GPT-4o rewrites each one in LinkedIn-native format — punchy opening line, short paragraphs with white space, personal angle, engagement question at the end. Schedules each post through the Buffer API for the next week. Updates Notion status to "Scheduled". Posts a weekly preview digest to Slack.

## Tools Used
- **Orchestration:** n8n
- **Content Source:** Notion (content calendar DB)
- **AI Rewriting:** OpenAI GPT-4o
- **Scheduling:** Buffer API
- **Status Update:** Notion
- **Preview:** Slack
- **Schedule:** Every Sunday 8 PM

## LinkedIn format rules GPT-4o follows

- First line: pattern interrupt — no "I'm excited to share"
- Max 2 lines per paragraph
- Empty line between every paragraph
- Personal story or real observation woven in
- Ends with a question to drive comments
- 150-300 words (LinkedIn sweet spot)
- Max 3 hashtags, no spam

## Content flow

```
Sunday 8 PM
  → Fetch "Approved" items from Notion (max 5)
  → For each: GPT-4o rewrites in LinkedIn format
  → Schedule via Buffer (next business day, 8 AM)
  → Update Notion: Approved → Scheduled
  → Post weekly digest to Slack
```

## Notion content calendar structure

| Field | Type | Values |
|---|---|---|
| Name | Title | Post title |
| Content | Text | Raw content/notes |
| Content Type | Select | insight/story/tip/announcement |
| Status | Select | Draft → Approved → Scheduled → Posted |
| Platform | Select | LinkedIn/Twitter/Instagram |

## Why I built this

LinkedIn engagement requires consistent posting — 3-5x per week minimum for growth. Writing in the right format every time was the bottleneck. Now I write raw notes during the week, approve them, and this workflow handles the rest on Sunday night.

## Setup

1. Notion integration + Content Calendar DB ID
2. OpenAI API key
3. Buffer account + API token + LinkedIn Profile ID
4. Slack Bot Token + #content channel
