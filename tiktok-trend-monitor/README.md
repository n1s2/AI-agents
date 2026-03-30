# FLOOWBOX - TikTok Trend Monitor and Script Writer

TikTok trends move fast — what works this week is irrelevant next week. This workflow monitors what's trending every morning and writes ready-to-record scripts before the trend peaks.

## What it does

Every morning at 7 AM, Perplexity AI searches for the current week's trending TikTok formats, sounds, and topics specifically within the configured niche. GPT-4o then writes 3 complete video scripts using those trends — each with a pattern-interrupt hook, fast-paced body, on-screen text suggestions, trending sound recommendation, and CTA. Scripts save to Notion and a daily briefing goes to Slack.

## Tools Used
- **Orchestration:** n8n
- **Trend Research:** Perplexity AI Sonar (real-time web search)
- **Script Writing:** OpenAI GPT-4o
- **Storage:** Notion content calendar
- **Briefing:** Slack
- **Schedule:** Daily 7 AM

## Flow

```
7 AM daily
  → Set niche + brand config
  → Perplexity: search trending TikTok content this week
  → GPT-4o writes 3 niche-specific scripts
  → Save to Notion (draft status)
  → Post daily briefing to Slack
```

## Script structure per video

```json
{
  "trend_used": "POV: you discovered automation",
  "hook": "POV: I stopped doing 80% of my admin work (not clickbait)",
  "body": "Last month I was spending 3 hours a day on emails, invoices, and follow-ups...",
  "cta": "Follow if you want to see how I did it",
  "on_screen_text": ["Before: 3 hours/day", "After: 20 minutes"],
  "sound_suggestion": "Trending lo-fi beat — check 'For You' page",
  "hashtags": ["#automation", "#founder", "#AItools"],
  "estimated_viral_potential": "high"
}
```

## Why I built this

TikTok content for FLOOWBOX was inconsistent because ideation took too long. By the time I figured out what to post, the trend had passed. Now I wake up to 3 scripts that are already trend-aligned. Record time dropped from "whenever I have ideas" to every day.

## Setup

1. Perplexity API key (Header Auth: `Authorization: Bearer YOUR_KEY`)
2. OpenAI API key
3. Notion integration + Content Calendar DB ID
4. Slack Bot Token + #content channel
5. Update niche, brand name, and audience in Set Niche Config node
