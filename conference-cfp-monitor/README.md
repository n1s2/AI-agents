# FLOOWBOX - Conference CFP Monitor (AI/ML Deadlines)

A single workshop paper publication transforms an MS application from "strong" to "published researcher." This workflow tracks every AI/ML conference CFP deadline weekly — including the workshop and demo tracks that are realistic targets for first publications.

## What it does

Every Monday, runs parallel searches for top conference CFPs (NeurIPS, ICML, ICLR, ACL, EMNLP, AAAI) and workshop/demo track CFPs specifically matching the research topics. GPT-4o assesses topic match, submission difficulty, and identifies the best venues for a first publication. Prioritizes urgent deadlines and provides a submission strategy. Updates a Notion CFP calendar and posts to Slack.

## Tools Used
- **Orchestration:** n8n
- **CFP Research (x2):** Perplexity AI Sonar (parallel)
- **Prioritization:** OpenAI GPT-4o
- **Calendar:** Notion
- **Alert:** Slack
- **Schedule:** Weekly Monday 9 AM

## Why workshop papers matter for admissions

Full papers at NeurIPS, ICML, and ICLR are extremely competitive. But workshop papers, demo papers, and extended abstracts at the same venues are much more achievable and count as genuine publications. MBZUAI and Stanford admissions committees value any peer-reviewed publication, regardless of format.

## Difficulty tiers

| Tier | Examples | Acceptance Rate |
|---|---|---|
| Entry | Workshop papers, demo tracks | 30-50% |
| Intermediate | Short papers, findings tracks | 20-35% |
| Competitive | Full papers at top venues | 10-25% |

## CFP output format

```json
{
  "urgent_cfps": [
    {
      "venue": "ICML 2026 Workshop on Efficient LLMs",
      "type": "workshop paper",
      "deadline": "May 15 2026",
      "topics_match": "high",
      "difficulty": "entry"
    }
  ],
  "best_for_first_publication": [
    {"venue": "EMNLP 2026 Demo Track", "reason": "Demo papers need working system not novel theory — n8n workflows qualify"}
  ],
  "submission_strategy": "Target one workshop paper at ICML or NeurIPS 2026 — deadline May/June. The multi-agent research workflow and RAG verification pipeline are both publishable as system demos."
}
```

## Personal note

This workflow is built for my own MS application prep. The RAG source verification agent and multi-agent research team in this repo are both publishable as demo papers at workshops focused on LLM systems and agent frameworks.

## Setup

1. Perplexity API key
2. OpenAI API key
3. Notion integration + CFP Calendar DB
4. Slack Bot Token + #ms-applications channel
5. Update research topics in Set CFP Config to match your work
