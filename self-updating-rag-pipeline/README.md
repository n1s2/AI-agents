# FLOOWBOX - Self-Updating RAG Knowledge Base

Most RAG systems have a stale knowledge problem — the vector database only reflects what was indexed at setup time. This pipeline automatically ingests new documents the moment they are added, keeping the knowledge base current without any manual re-indexing.

## What it does

Accepts any document via webhook — either a URL or raw text. If a URL, Jina AI fetches the full content. The text is chunked into 500-word segments with 50-word overlap to preserve context across boundaries. OpenAI generates embeddings for each chunk using `text-embedding-3-small`. Chunks are upserted directly into Qdrant with metadata. Every ingestion logs to Google Sheets for audit.

## Tools Used
- **Orchestration:** n8n
- **Content Fetching:** Jina AI reader API
- **Chunking:** Custom JS code node (500 words, 50 overlap)
- **Embeddings:** OpenAI text-embedding-3-small
- **Vector DB:** Qdrant (self-hosted)
- **Audit Log:** Google Sheets
- **Trigger:** Webhook

## Flow

```
New document arrives (webhook)
  → Extract: URL or raw text + metadata
  → IF URL: Jina AI fetches content
  → Merge content + metadata
  → Chunk: 500 words, 50-word overlap
  → For each chunk: OpenAI embedding
  → Upsert to Qdrant with payload
  → Log ingestion to Google Sheets
  → Return: status + chunk count
```

## Why 500-word chunks with 50-word overlap

Chunks too large lose retrieval precision. Chunks too small lose context. 500 words hits the sweet spot for semantic search — specific enough to be relevant, long enough to answer a question. The 50-word overlap ensures sentences that straddle chunk boundaries are captured in both adjacent chunks.

## Webhook payload

```json
{
  "url": "https://docs.example.com/api-reference",
  "title": "API Reference v2",
  "type": "documentation",
  "source": "notion_sync"
}
```

OR with raw text:

```json
{
  "text": "Full document content here...",
  "title": "Meeting Notes March 2026",
  "type": "internal",
  "source": "manual"
}
```

## Why I built this

A client's support chatbot was giving outdated answers because the knowledge base was only updated quarterly. Connecting this ingestion webhook to their CMS meant every time they published a new help article, the chatbot knew about it within minutes.

## Qdrant collection setup

```bash
curl -X PUT http://localhost:6333/collections/knowledge_base \
  -H 'Content-Type: application/json' \
  -d '{"vectors": {"size": 1536, "distance": "Cosine"}}'
```

## Setup

1. Qdrant running locally or on cloud (qdrant.io)
2. OpenAI API key (embeddings + auth header)
3. Jina AI API key
4. Google Sheets ID for ingestion log
