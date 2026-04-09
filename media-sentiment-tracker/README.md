# FLOOWBOX - Media Sentiment Tracker

Understanding how the media portrays your industry and competitors — across tech press, business media, and social channels — is critical intelligence. This workflow aggregates and quantifies it every week.

## What it does

Every Friday at 4 PM, three Perplexity searches run simultaneously scanning tech media, business and financial media, and social signals for sentiment toward the AI automation industry and specific brands. GPT-4o synthesizes all three into a unified sentiment score, tracks week-over-week change, identifies viral moments and reputational risks, and writes a narrative summary of the week's media story. Historical trend logs to Google Sheets for long-term tracking.

## Tools Used
- **Orchestration:** n8n
- **Media Scanning (x3):** Perplexity AI Sonar (parallel)
- **Synthesis:** OpenAI GPT-4o
- **Trend Logging:** Google Sheets
- **Report:** Slack
- **Schedule:** Weekly Friday 4 PM

## Three scanning streams

| Stream | Sources |
|---|---|
| Tech Media | TechCrunch, Wired, The Verge, Ars Technica |
| Business Media | Bloomberg, Forbes, WSJ, Financial Times |
| Social Signals | Twitter/X, LinkedIn, Hacker News, Reddit |

## Weekly sentiment report

```json
{
  "overall_sentiment": "positive",
  "sentiment_score": 7.2,
  "brand_sentiments": [
    {"brand": "OpenAI", "sentiment": "mixed", "key_story": "GPT-5 rumors vs safety criticism"},
    {"brand": "n8n", "sentiment": "very positive", "key_story": "Strong developer community growth"}
  ],
  "viral_moments": ["Anthropic Constitutional AI paper widely shared on HN"],
  "week_over_week_change": "improving",
  "reputational_risks": ["Ongoing EU AI Act compliance uncertainty"],
  "weekly_narrative": "Positive week for AI automation tools overall, driven by strong product releases..."
}
```

## Why I built this

FLOOWBOX operates in the AI automation space — understanding how the narrative is shifting helps anticipate client concerns, sales objections, and market opportunities. When media sentiment toward AI automation turns negative, clients get cold feet. Tracking it weekly means being prepared for those conversations.

## Historical tracking value

Google Sheets logs weekly scores over time — after 3 months you can see sentiment trends that are invisible week to week. Useful for identifying whether industry reputation is improving or deteriorating over longer periods.

## Setup

1. Perplexity API key
2. OpenAI API key
3. Google Sheets ID (for historical trend)
4. Slack Bot Token + #media-intel channel
5. Update brands in Set Tracking Config
