# FLOOWBOX - Reddit Mention Monitor and Alert

Reddit is where buyers talk honestly about their problems and tools. This workflow monitors relevant subreddits hourly and alerts when there is a real opportunity to engage — with a suggested reply already written.

## What it does

Searches Reddit every hour for brand mentions and industry keywords. Filters for posts with meaningful engagement. GPT-4o classifies each mention — direct brand mention, competitor comparison, industry question, lead opportunity, or negative sentiment. Scores each on opportunity value 0-10. Only high-scoring mentions trigger a Slack alert, including an AI-written reply suggestion that is helpful and non-promotional.

## Tools Used
- **Orchestration:** n8n
- **Data:** Reddit API (public JSON endpoint)
- **AI Classification:** OpenAI GPT-4o
- **Alerts:** Slack (#brand-mentions)
- **Schedule:** Hourly

## Flow

```
Every hour
  → Search Reddit for brand keywords (last 24h, sorted by new)
  → Split into individual posts
  → Filter: score >= 2 (remove noise)
  → GPT-4o: classify + score opportunity
  → Filter: opportunity_score >= 5
  → Slack alert with suggested reply
```

## Mention types classified

| Type | Example | Action |
|---|---|---|
| `direct_brand` | "Anyone used FLOOWBOX?" | Respond immediately |
| `lead_opportunity` | "Need help automating invoices" | Offer solution |
| `competitor_comparison` | "Zapier vs n8n?" | Engage helpfully |
| `industry_question` | "How do you automate outreach?" | Share expertise |
| `negative` | Complaint about similar tool | Monitor only |

## Example Slack alert

```
Reddit Mention — lead_opportunity

Subreddit: r/entrepreneur
Title: "How do I stop manually sending follow-up emails?"
Score: 47 | Sentiment: neutral
Urgency: immediate

Suggested Reply:
"I had the same problem. Built a workflow in n8n that automatically sends
follow-ups based on whether someone opened the previous email.
Happy to share the setup if useful."
```

## Why I built this

Reddit drove 3 of FLOOWBOX's first client conversations. But monitoring manually meant checking 6 subreddits multiple times a day. This catches every relevant post within an hour and gives a ready-to-post reply — so engaging takes 30 seconds instead of 10 minutes of research.

## Setup

1. Update keywords array in Set Brand Keywords node
2. Add/remove subreddits as needed
3. OpenAI API key
4. Slack Bot Token + #brand-mentions channel
