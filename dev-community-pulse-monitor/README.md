# FLOOWBOX - Developer Community Pulse Monitor

What developers are complaining about, excited by, and asking to learn — across Hacker News, Reddit, and Dev.to simultaneously — is the clearest signal of where the ecosystem is heading. This workflow captures that pulse every week.

## What it does

Every Friday, pulls live data from three developer platforms in parallel — Hacker News top stories via Firebase API, Reddit top posts from ML/AI/n8n subreddits, and Dev.to trending AI articles. GPT-4o synthesizes all three into a weekly community pulse: hottest topics, developer concerns, tools being discussed, tutorials in demand, and emerging controversies. Identifies "what to build" opportunities based on what developers are asking for.

## Tools Used
- **Orchestration:** n8n
- **Hacker News:** Firebase HN API (live)
- **Reddit:** Reddit JSON API
- **Dev.to:** Dev.to REST API
- **Synthesis:** OpenAI GPT-4o
- **Report:** Slack
- **Schedule:** Weekly Friday 9 AM

## Three live data sources

| Source | Data |
|---|---|
| Hacker News | Top 5 stories (real-time scores) |
| Reddit | Top posts from r/MachineLearning + r/LocalLLaMA + r/n8n |
| Dev.to | Top 10 trending AI articles this week |

## Pulse output

```json
{
  "hottest_topics": ["LangGraph state machines", "Local LLM fine-tuning", "Agent memory patterns"],
  "developer_concerns": ["LLM cost at scale", "Hallucination in production"],
  "tutorials_in_demand": ["Building agents with memory", "RAG chunking strategies"],
  "controversies": ["GPT-4o quality degradation debate"],
  "what_to_build_based_on_demand": [
    "n8n workflow with persistent agent memory",
    "Cost-tracking layer for LLM API calls"
  ],
  "community_mood": "excited"
}
```

## Why I built this

The "tutorials in demand" field alone makes this valuable — it directly tells you what content to create, what problems to build solutions for, and what FLOOWBOX clients are likely to ask about next week. The dev community is 2-3 months ahead of the mainstream market.

## Setup

1. OpenAI API key
2. Slack Bot Token + #dev-trends channel
3. No API keys needed for HN, Reddit (public), or Dev.to (public)
