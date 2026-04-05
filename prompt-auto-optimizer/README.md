# FLOOWBOX - Prompt Auto-Optimizer Agent

Most prompts are written by intuition and never systematically improved. This workflow analyzes any prompt, identifies its weaknesses, generates three improved variants using different optimization strategies, and tests the best one live.

## What it does

Send a prompt and describe the goal. GPT-4o analyzes the original prompt for weaknesses — ambiguity, missing context, no format specification, no role definition, weak instructions. Then generates three improved variants: one structured with explicit output format, one with role definition and few-shot examples, one using chain-of-thought reasoning. Tests the recommended variant against the actual model and returns the output alongside all three versions.

## Tools Used
- **Orchestration:** n8n
- **Analysis:** OpenAI GPT-4o (weakness identification)
- **Generation:** OpenAI GPT-4o (3 optimized variants)
- **Testing:** OpenAI API (live test of best variant)
- **Library:** Notion
- **Trigger:** Webhook

## Flow

```
POST: {prompt, goal, model, iterations}
  → GPT-4o: analyze weaknesses + score original (0-10)
  → GPT-4o: generate 3 variants + predict scores
  → Test recommended variant live
  → Save all to Notion prompt library
  → Return: variants + test output + scores
```

## Three optimization strategies

| Version | Strategy | Best for |
|---|---|---|
| V1 | Structured + output format | Tasks needing consistent JSON/structured output |
| V2 | Role + few-shot examples | Creative or stylistic tasks |
| V3 | Chain-of-thought | Complex reasoning tasks |

## Example improvement

**Original prompt (score: 3/10):**
```
Summarize this article.
```

**V1 — Structured (predicted score: 8/10):**
```
You are a professional editor. Summarize the following article for a 
business executive audience.

Requirements:
- Length: 3-4 sentences maximum
- Include: main finding, business implications, one key statistic
- Tone: formal, third person
- Format: plain paragraph, no bullet points

Article: [ARTICLE]
```

## Why this matters for AI research

Prompt engineering is a core skill in applied NLP research. Systematic prompt optimization using LLMs as critics connects directly to research on automatic prompt engineering (APE), self-refinement, and Constitutional AI feedback mechanisms studied at MBZUAI and Stanford.

## Setup

1. OpenAI API key
2. Notion integration + Prompt Library DB
3. POST to webhook with prompt + goal
