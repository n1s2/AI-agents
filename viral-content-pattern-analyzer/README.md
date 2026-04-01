# FLOOWBOX - Viral Content Pattern Analyzer

The best content creators do not guess what works — they study what is already going viral and reverse-engineer the patterns. This workflow does that research automatically every week.

## What it does

Every Friday morning, searches for the top viral posts of the week across LinkedIn, Twitter, and Reddit in the AI automation niche using Perplexity. GPT-4o analyzes all viral content together and extracts repeatable patterns — hook structures that appeared in multiple high-performing posts, topic angles, emotional triggers, and format types. Generates 3-5 ready-to-use content ideas for FLOOWBOX based on what worked that week. Saves full analysis to Notion and posts a briefing to Slack.

## Tools Used
- **Orchestration:** n8n
- **Viral Research:** Perplexity AI Sonar (LinkedIn + Twitter)
- **Reddit Data:** Reddit public API
- **Pattern Analysis:** OpenAI GPT-4o
- **Storage:** Notion
- **Briefing:** Slack
- **Schedule:** Weekly Friday 9 AM

## Flow

```
Friday 9 AM
  → Parallel search:
      Perplexity: top viral LinkedIn posts this week
      Perplexity: top viral Twitter posts this week
      Reddit API: top posts from relevant subreddits
  → GPT-4o: extract patterns + generate ideas
  → Save to Notion
  → Post Slack briefing
```

## Pattern types extracted

```json
{
  "top_hook_patterns": [
    {
      "pattern": "Quantified transformation",
      "example": "I went from 40 hours/week to 4 hours/week in 90 days",
      "why_it_works": "Specific numbers make the claim credible and desirable"
    }
  ],
  "content_ideas_for_floowbox": [
    {
      "idea": "Show before/after of a client's manual process vs automated",
      "hook": "This is what 3 hours of manual work looks like automated",
      "format": "carousel",
      "platform": "LinkedIn"
    }
  ],
  "weekly_insight": "Story-based hooks with specific time/money numbers dominated this week"
}
```

## Why I built this

Content strategy based on gut feel is inconsistent. This workflow means every week's content plan is informed by what actually went viral in the niche that week — not what worked 6 months ago or what I feel like posting.

## Setup

1. Perplexity API key
2. OpenAI API key
3. Notion integration + Content Calendar DB ID
4. Slack Bot Token + #content channel
