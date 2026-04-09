# FLOOWBOX - Fake News Classifier

Misinformation spreads faster than corrections. This workflow runs a three-pass verification pipeline on any article — credibility signals, live cross-referencing, and final verdict — before you share it.

## What it does

Send any article URL or text. Jina AI fetches the full content. A three-pass pipeline runs: Pass 1 uses GPT-4o to analyze credibility signals (headline accuracy, source quality, emotional language, anonymous sources) and extract specific checkable claims. Pass 2 uses Perplexity to cross-reference those claims against current sources and check for existing fact-checker verdicts. Pass 3 uses GPT-4o to issue a final verdict with confidence score and sharing recommendation.

## Tools Used
- **Orchestration:** n8n
- **Content Fetch:** Jina AI
- **Pass 1:** OpenAI GPT-4o (credibility signals)
- **Pass 2:** Perplexity AI (live cross-referencing)
- **Pass 3:** OpenAI GPT-4o (final verdict)
- **Logging:** Google Sheets
- **Trigger:** Webhook

## Three-pass pipeline

```
Article URL/text
  → Jina AI: fetch full content
  → Pass 1: GPT-4o credibility signals + extract claims
  → Pass 2: Perplexity cross-reference claims vs current sources
  → Pass 3: GPT-4o final verdict + confidence + sharing recommendation
  → Log to Google Sheets
```

## Verdict scale

| Verdict | Meaning |
|---|---|
| LIKELY TRUE | Multiple credible sources confirm |
| MOSTLY TRUE | Core accurate, minor details off |
| MIXED | Some accurate, some inaccurate |
| MOSTLY FALSE | Core claim not supported |
| LIKELY FALSE | Contradicted by credible sources |
| SATIRE | Satirical content mistaken for news |
| UNVERIFIABLE | Cannot be confirmed or denied |

## Example output

```json
{
  "verdict": "MOSTLY FALSE",
  "confidence": "high",
  "credibility_score": 2.5,
  "summary": "The headline claim about X is not supported by official data. The statistic cited appears to be from a 3-year-old study that has been superseded.",
  "share_recommendation": "do not share"
}
```

## Why I built this

WhatsApp forwards and social media posts regularly contain misinformation that spreads through professional networks. This gives a quick automated check before forwarding anything — reducing the chance of sharing false information.

## Setup

1. Jina AI API key
2. OpenAI API key
3. Perplexity API key
4. Google Sheets ID
