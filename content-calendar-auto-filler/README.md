# FLOOWBOX - Content Calendar Auto-Filler

Starting the week with an empty content calendar kills consistency. This workflow fills the entire week every Sunday evening — 10 posts across LinkedIn and Twitter, trend-informed and ready for review.

## What it does

Every Sunday at 7 PM, fetches the week's trending topics in the AI automation and founder space. Checks Notion for empty calendar slots. GPT-4o generates a full week of content — 5 days × 2 posts each (LinkedIn and Twitter/X) — with a coherent week theme, at least 3 posts tied to current trending topics, and specific hooks, key points, CTAs, and hashtags for each. Creates a Notion entry per post and posts Monday's hooks to Slack as a preview.

## Tools Used
- **Orchestration:** n8n
- **Trend Research:** Perplexity AI Sonar
- **Content Generation:** OpenAI GPT-4o
- **Calendar:** Notion (one page per post)
- **Preview:** Slack
- **Schedule:** Weekly Sunday 7 PM

## Output structure per post

```json
{
  "day": "Tuesday",
  "platform": "LinkedIn",
  "topic": "Why most automation projects fail in month 3",
  "angle": "Anti-advice — what NOT to do",
  "hook": "I watched 6 automation projects die at exactly the same moment.",
  "key_points": ["They automated before documenting", "No one owned the workflow", "Trigger failure went unnoticed for weeks"],
  "cta": "What killed your last automation attempt?",
  "hashtags": ["automation", "founder", "n8n"],
  "trend_connection": "Ties to this week's discussion about AI project failure rates"
}
```

## Why I built this

FLOOWBOX's content was inconsistent — posting 4 times one week, nothing the next — because planning happened reactively. This workflow means I wake up Monday with a full week planned and only need to write the actual post copy based on the briefs. Content consistency went from "whenever I had time" to every weekday.

## Setup

1. Perplexity API key
2. OpenAI API key
3. Notion integration + Content Calendar DB (with Platform, Status, Day, Hook, CTA fields)
4. Slack Bot Token + #content channel
