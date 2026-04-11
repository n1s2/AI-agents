# FLOOWBOX - GitHub Trending Repo Analyzer

What developers are building on GitHub today predicts what will be in production systems in 6 months. This workflow monitors trending repositories daily and extracts what matters for AI and automation engineers.

## What it does

Every morning, queries GitHub Search API for the top 20 recently created repositories gaining stars in AI, LLM, automation, and agents topics. Simultaneously runs a Perplexity search for context on what is gaining traction this week. GPT-4o analyzes both sources to identify emerging technologies, patterns in what is trending, repos worth starring, learning opportunities, and potential n8n integration ideas. Daily Slack brief and Notion save.

## Tools Used
- **Orchestration:** n8n
- **GitHub Data:** GitHub Search API
- **Context:** Perplexity AI Sonar
- **Analysis:** OpenAI GPT-4o
- **Storage:** Notion
- **Brief:** Slack
- **Schedule:** Daily 8 AM

## Flow

```
8 AM daily
  → GitHub API: top 20 repos (AI/LLM/automation, sorted by stars)
  → Perplexity: what is gaining traction this week
  → GPT-4o: extract insights + patterns + opportunities
  → Save to Notion
  → Post Slack brief
```

## Analysis output

```json
{
  "emerging_technologies": ["CrewAI multi-agent patterns", "LLM routing frameworks"],
  "patterns_in_trending": ["Shift from single-agent to multi-agent architectures"],
  "repos_worth_starring": ["microsoft/autogen v2", "langchain-ai/langgraph"],
  "learning_opportunities": [
    {"repo": "langchain-ai/langgraph", "what_to_learn": "State machine patterns for agent workflows"}
  ],
  "potential_integrations_with_n8n": [
    "LangGraph state management could enhance n8n's agent memory patterns"
  ]
}
```

## Why I built this

Staying current with what is trending on GitHub is one of the fastest ways to identify emerging tools before they become mainstream. This workflow surfaces what the dev community is excited about — before it shows up in job descriptions or blog posts.

## Setup

1. GitHub Personal Access Token (read-only, for higher rate limits)
2. Perplexity API key
3. OpenAI API key
4. Notion integration + DB ID
5. Slack Bot Token + #dev-trends channel
