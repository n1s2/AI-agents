# FLOOWBOX - Stack Overflow Question Miner

Stack Overflow questions are a direct map of developer pain points. This workflow mines them daily to identify what problems people are struggling with, which unanswered questions represent authority-building opportunities, and what content would answer the most questions at once.

## What it does

Every day, queries the Stack Overflow API for recent questions tagged with n8n, langchain, openai-api, llm, and ai-agents. GPT-4o analyzes the patterns — identifying top pain points, finding the highest-voted unanswered questions (opportunities to answer and build reputation), spotting recurring confusion that suggests documentation gaps, and generating content ideas that would address the most questions simultaneously. Slack alert with the best questions to answer that day.

## Tools Used
- **Orchestration:** n8n
- **Data:** Stack Overflow API v2.3
- **Analysis:** OpenAI GPT-4o
- **Storage:** Notion (content ideas)
- **Alert:** Slack
- **Schedule:** Daily 9 AM

## Flow

```
9 AM daily
  → SO API: questions tagged n8n/langchain/openai-api (last 7 days)
  → SO API: n8n questions (last 3 days, by activity)
  → Deduplicate + process
  → GPT-4o: analyze pain points + opportunities + content gaps
  → Save content ideas to Notion
  → Slack: best questions to answer today
```

## Analysis output

```json
{
  "top_pain_points": [
    "n8n webhook authentication confusion",
    "LangChain memory persistence across sessions"
  ],
  "best_questions_to_answer": [
    {
      "title": "How to pass context between n8n workflow executions?",
      "votes": 12,
      "why_valuable": "High votes, no accepted answer, directly in FLOOWBOX expertise area"
    }
  ],
  "content_ideas": [
    {"title": "n8n session management: complete guide", "would_answer_n_questions": 8}
  ],
  "documentation_gaps": ["n8n error handling for webhook timeouts is poorly documented"]
}
```

## Why I built this

Answering Stack Overflow questions in your specialty area builds domain authority and drives inbound interest. This workflow makes it systematic — every morning I know exactly which questions are worth spending 15 minutes on. The content ideas from recurring confusion patterns directly feed the FLOOWBOX blog calendar.

## Stack Exchange API

Uses the public Stack Exchange API — no authentication needed for read operations, though an API key increases rate limits.

## Setup

1. OpenAI API key
2. Notion integration + Content Calendar DB
3. Slack Bot Token + #content channel
4. Optional: Stack Exchange API key for higher rate limits
