# FLOOWBOX - AI Response Fact-Checker

LLMs hallucinate. Before any AI-generated content goes live, factual claims should be verified against real sources. This workflow extracts every verifiable claim from a text and checks each one independently against current web data.

## What it does

Accepts any text — an AI-generated article, a research summary, a chatbot response. GPT-4o extracts all specific, verifiable factual claims (statistics, dates, named events, scientific claims). Each claim gets independently verified via Perplexity AI which searches current web sources and returns TRUE, PARTIALLY TRUE, or FALSE with a brief explanation. GPT-4o compiles all results into a final accuracy report with an overall trust verdict.

## Tools Used
- **Orchestration:** n8n
- **Claim Extraction:** OpenAI GPT-4o
- **Verification:** Perplexity AI Sonar (per-claim web search)
- **Report:** OpenAI GPT-4o
- **Logging:** Google Sheets
- **Trigger:** Webhook

## Flow

```
POST: {text, context, domain}
  → GPT-4o: extract all verifiable factual claims
  → For each claim: Perplexity web search → TRUE/FALSE/PARTIAL
  → Aggregate all verdicts
  → GPT-4o: compile accuracy report + trust score
  → Log to Google Sheets
  → Return full report
```

## Per-claim verification output

```json
{
  "claim": "GPT-4 has 1.8 trillion parameters",
  "verdict": "PARTIALLY TRUE",
  "explanation": "The 1.8 trillion figure is from unconfirmed leaks. OpenAI has not disclosed official parameter count..."
}
```

## Final report

```json
{
  "overall_accuracy": "medium",
  "accuracy_score": 6.5,
  "true_claims": 4,
  "false_claims": 1,
  "partially_true": 2,
  "trust_verdict": "questionable",
  "false_claim_details": [
    {"claim": "OpenAI was founded in 2013", "correction": "OpenAI was founded in December 2015"}
  ],
  "summary": "7 claims checked. 4 verified accurate, 1 false, 2 partially accurate. Main concern is the founding date error."
}
```

## Why this matters

Factual accuracy in AI outputs is one of the most critical open problems in NLP research. This workflow implements an automated fact-checking pipeline — a practical application of retrieval-augmented verification. Directly relevant to AI reliability and hallucination reduction research at MBZUAI and Stanford.

## Use cases

- Verify AI-generated blog posts before publishing
- Check chatbot responses in production
- Audit research summaries
- Verify news claims before sharing
- Quality gate for content pipelines

## Setup

1. OpenAI API key
2. Perplexity API key
3. Google Sheets ID
