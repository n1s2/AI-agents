# FLOOWBOX - LLM Output Quality Evaluator

Before deploying any LLM-powered feature to production, outputs need to be evaluated systematically. This workflow implements the LLM-as-Judge pattern — using multiple specialized evaluator agents to score responses across three independent quality dimensions.

## What it does

Accepts a prompt-response pair and runs three evaluator agents in parallel — each scoring a different quality dimension independently. An Accuracy agent checks factual correctness and hallucinations. A Completeness agent measures coverage and relevance. A Safety agent flags harmful content and bias. All scores aggregate into a final percentage and PASS/PARTIAL/FAIL verdict. Logs every evaluation to Google Sheets for trend tracking.

## Tools Used
- **Orchestration:** n8n
- **Evaluation Agents:** OpenAI GPT-4o (3 parallel, specialized)
- **Logging:** Google Sheets
- **Trigger:** Webhook

## Three evaluation dimensions

| Agent | Evaluates | Key signals |
|---|---|---|
| Accuracy Agent | Factual correctness | Hallucinations, wrong facts, correct elements |
| Completeness Agent | Coverage and relevance | Missing elements, irrelevant content, coverage % |
| Safety Agent | Harm and bias | Harmful content, bias detected, tone |

## Flow

```
POST: {prompt, response, task_type, expected_output}
  → 3 evaluator agents run in parallel
  → Each returns score/10 + verdict + findings
  → Aggregate all scores
  → Compute final score % + PASS/PARTIAL/FAIL
  → Log to Google Sheets
  → Return full evaluation JSON
```

## Evaluation output

```json
{
  "score_percent": 78,
  "overall_verdict": "PARTIAL",
  "dimension_scores": [
    {"dimension": "accuracy", "score": 8, "verdict": "pass"},
    {"dimension": "completeness", "score": 6, "verdict": "partial"},
    {"dimension": "safety", "score": 10, "verdict": "pass"}
  ],
  "issues": [
    "Missing discussion of edge cases",
    "No mention of error handling"
  ]
}
```

## Why this matters for AI research

This implements **LLM-as-Judge** — a core evaluation methodology in AI safety and alignment research. Papers like MT-Bench and LMSYS Chatbot Arena use variants of this pattern. Understanding multi-dimensional evaluation is directly relevant to MBZUAI and Stanford AI research programs.

## Use cases

- Quality gate before publishing LLM responses to users
- A/B testing two model versions
- Monitoring production LLM quality over time
- Evaluating fine-tuned model outputs

## Setup

1. OpenAI API key
2. Google Sheets ID for evaluation log
3. Webhook configured in your LLM application
