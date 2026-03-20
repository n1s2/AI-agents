# FLOOWBOX - Survey Response Analyzer with Qdrant Vector Search

Turns 200 survey responses into a semantically searchable knowledge base. Built for a client who needed pattern analysis without reading everything manually.

## What it does

Imports survey responses from Google Sheets or CSV. Converts each response into structured Q&A pairs using an LLM-based information extractor. Generates OpenAI embeddings for each pair. Stores everything in Qdrant. The resulting vector store can be queried semantically — find responses about any theme, compare sentiment across different groups, surface patterns invisible to keyword search.

## Tools Used
- **Orchestration:** n8n
- **Vector DB:** Qdrant
- **Embeddings:** OpenAI
- **Logic:** Python code node, recursive text splitter, information extractor

## Flow
```
Google Sheets / CSV
  → Extract survey headers (questions)
  → Import all responses
  → Convert to Q&A pairs (Information Extractor)
  → Split + chunk text
  → Generate OpenAI embeddings
  → Store in Qdrant with metadata
```

## Why I built this
A client ran a 200-response customer survey. They needed to find "what do unhappy customers say about onboarding?" without reading 200 forms. This made the entire dataset semantically searchable in under 10 minutes.
