# FLOOWBOX - Hashtag Research Automator

Using the wrong hashtags means your posts reach nobody. This workflow researches current hashtags with real post counts, builds three rotation sets sized for your account, and delivers them copy-paste ready.

## What it does

Set a topic, platform, account size, and content type. Perplexity searches for current hashtags across all size tiers with actual post count data. GPT-4o builds three rotation sets — mixing mega, large, medium, and niche hashtags in the right proportions for the account size. Rotating sets prevents Instagram shadowbanning. All sets save to a Notion library. Set A arrives in Slack copy-paste ready.

## Tools Used
- **Orchestration:** n8n
- **Hashtag Research:** Perplexity AI Sonar (real-time data)
- **Set Building:** OpenAI GPT-4o
- **Library:** Notion
- **Output:** Slack (copy-paste ready)
- **Trigger:** Manual (run per topic)

## Flow

```
Manual trigger
  → Set topic + platform + account size
  → Perplexity: research 30 hashtags with post counts
  → GPT-4o: build 3 rotation sets + strategy
  → Save to Notion hashtag library
  → Format copy-paste output
  → Post Set A to Slack
```

## Output structure

```json
{
  "strategy_note": "Small accounts should lead with niche hashtags for better reach rate",
  "sets": [
    {
      "set_name": "Set A",
      "hashtags": ["#aiautomation", "#n8n", "#nocode", ...],
      "mix": "3 mega + 7 large + 8 medium + 7 niche",
      "total_reach": "~45M combined posts"
    }
  ],
  "trending_now": ["#AItools2026", "#automateeverything"],
  "hashtags_to_avoid": ["#automation (oversaturated, 50M+ posts)"]
}
```

## Why I built this

A content client was using the same 30 hashtags on every post — no rotation, no size variety, all mega hashtags. Reach was declining monthly. After switching to researched, rotated sets with proper niche mix, average reach increased 3x within 3 weeks.

## Sizing guide GPT-4o follows

| Account Size | Mega | Large | Medium | Niche |
|---|---|---|---|---|
| Small (<5k) | 3 | 5 | 8 | 9 |
| Medium (5-50k) | 5 | 8 | 8 | 4 |
| Large (50k+) | 8 | 10 | 7 | 0 |

## Setup

1. Perplexity API key
2. OpenAI API key
3. Notion integration + Content Calendar DB
4. Slack Bot Token + #content channel
5. Update topic/platform in Set Post Topic node per use
