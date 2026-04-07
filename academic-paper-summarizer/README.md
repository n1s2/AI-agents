# FLOOWBOX - Academic Paper Summarizer (ArXiv + Semantic Scholar)

Keeping up with AI research means reading dozens of papers per week. This workflow turns any ArXiv paper into a structured summary in under 2 minutes — with separate versions for researchers and non-specialists.

## What it does

Send any ArXiv paper ID or URL. The workflow fetches structured metadata from the ArXiv API (title, authors, abstract, date) and fetches the full rendered HTML from ar5iv via Jina AI. GPT-4o generates a complete structured summary: one-sentence TL;DR, problem statement, methodology, key findings, limitations, and practical applications. Two summary lengths — plain English for non-specialists and technical for researchers. Saves to Notion research library with formatted citation.

## Tools Used
- **Orchestration:** n8n
- **Metadata:** ArXiv API (XML)
- **Full Text:** ar5iv HTML rendering + Jina AI
- **Summarization:** OpenAI GPT-4o
- **Library:** Notion
- **Trigger:** Webhook

## Flow

```
POST: {arxiv_id or url, audience, focus}
  → Parallel fetch:
      ArXiv API → title, authors, abstract, date
      Jina AI → full paper HTML content
  → Merge both sources
  → GPT-4o: structured summary
  → Save to Notion library
  → Return full summary JSON
```

## Summary structure

```json
{
  "tldr": "Proposes a new attention mechanism that reduces transformer memory by 60% with no accuracy loss on standard benchmarks",
  "problem_solved": "Standard attention scales quadratically with sequence length, making long-document transformers prohibitively expensive",
  "methodology": "Sparse attention pattern with learned routing — only top-k token pairs computed per layer",
  "key_findings": ["60% memory reduction", "Matches full attention on GLUE benchmarks", "Scales to 32k token contexts"],
  "limitations": ["Not tested on code generation", "Routing adds ~8% compute overhead"],
  "practical_applications": ["Long document summarization", "Code completion with large context windows"]
}
```

## Why I built this

Preparing for MS in AI applications requires staying current with research. Reading each paper fully takes 45-60 minutes. This workflow processes the paper in 90 seconds and saves the summary to a searchable Notion library I can reference in SOPs and interviews.

## Webhook payload

```json
{
  "arxiv_id": "2401.00001",
  "audience": "researcher",
  "focus": "methodology"
}
```

## Setup

1. Jina AI API key
2. OpenAI API key
3. Notion integration + Research Library DB
