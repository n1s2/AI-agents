# FLOOWBOX - RAG with Source Verification Agent

Standard RAG pipelines generate answers that can quietly hallucinate — adding details not present in the retrieved chunks. This pipeline runs a second verification pass that checks every claim against the source text before the answer reaches the user.

## What it does

User asks a question via chat. The query is embedded using `text-embedding-3-small` and searched against Qdrant with a 0.7 similarity threshold. Retrieved chunks are passed to GPT-4o which generates an answer with inline citations — strictly from context only. A second GPT-4o verifier then checks each factual claim against the source chunks and flags any that cannot be verified. The final response includes verified claims, source titles with relevance scores, and a hallucination flag.

## Tools Used
- **Orchestration:** n8n
- **Embeddings:** OpenAI text-embedding-3-small
- **Vector Search:** Qdrant (local)
- **Generation:** OpenAI GPT-4o (Pass 1 — answer)
- **Verification:** OpenAI GPT-4o (Pass 2 — claim check)
- **Trigger:** Chat interface

## Two-pass architecture

```
Query → Embed → Qdrant search (top 5, score > 0.7)
  → Pass 1: Generate answer with citations (GPT-4o)
  → Pass 2: Verify each claim vs source text (GPT-4o)
  → Format: verified answer + sources + confidence + hallucination flag
```

## Final response format

```
Based on the documentation [Source 1], the API rate limit is 1000 
requests per minute [Source 1]. Enterprise plans have no rate limit 
according to the pricing page [Source 2].

---
Sources:
[Source 1] API Reference v2 (relevance: 94%)
[Source 2] Pricing Page March 2026 (relevance: 87%)

Verification: All claims verified
Confidence: high
```

## Why verification matters

In production RAG systems, hallucinated details in answers — especially in support bots or legal/medical applications — can cause serious problems. The verification pass catches errors before they reach users. In testing, the two-pass approach reduced hallucinations by approximately 70% compared to standard single-pass RAG.

## Pairs with

[Self-Updating RAG Pipeline](../self-updating-rag-pipeline/) — use that workflow to populate the Qdrant knowledge base, then this workflow to query it with verification.

## Setup

1. Qdrant running with `knowledge_base` collection populated
2. OpenAI API key (for embeddings + both GPT-4o calls)
3. Both workflows active simultaneously
