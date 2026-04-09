# FLOOWBOX - News Aggregator and Bias Detector

The same story gets covered completely differently depending on who is telling it. This workflow pulls the same topics from left-leaning, right-leaning, and neutral outlets simultaneously — then surfaces the framing differences so you can read the actual news, not the spin.

## What it does

Every morning, three Perplexity searches run in parallel on the same topics — one querying progressive/left outlets, one querying conservative/right outlets, one querying wire services and neutral sources. GPT-4o compares all three to identify framing differences, stories covered only by one side, and bias patterns. Generates a single "most neutral summary" of what actually happened. Saves to Notion and posts a daily Slack digest.

## Tools Used
- **Orchestration:** n8n
- **Coverage (x3):** Perplexity AI Sonar (parallel, same-day filter)
- **Analysis:** OpenAI GPT-4o
- **Storage:** Notion
- **Digest:** Slack
- **Schedule:** Daily 7 AM

## What bias analysis reveals

```json
{
  "top_stories": [
    {
      "headline": "AI Regulation Bill Passes Committee",
      "left_angle": "Celebrates consumer protections, quotes civil society",
      "right_angle": "Warns of innovation stifling, quotes industry leaders",
      "neutral_summary": "Senate Commerce Committee passed AI oversight bill 12-8 on party lines"
    }
  ],
  "bias_patterns_detected": [
    {"outlet_type": "left", "pattern": "Tech regulation framed as consumer protection", "example": "..."},
    {"outlet_type": "right", "pattern": "Same regulation framed as government overreach", "example": "..."}
  ],
  "stories_only_covered_by_one_side": ["OpenAI whistleblower story — only in progressive outlets today"]
}
```

## Why I built this

Reading news from only one source creates a distorted view of what is happening. This workflow creates a daily brief that shows the same event from multiple perspectives — making it much easier to separate fact from framing.

## Setup

1. Perplexity API key
2. OpenAI API key
3. Notion integration + DB ID
4. Slack Bot Token + #news-brief channel
5. Update topics in Set Topics Config
