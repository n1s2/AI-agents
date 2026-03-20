# FLOOWBOX - RAG with Proper OpenAI Citation Formatting

OpenAI's file retrieval returns citations as `【4:0†source】` markers. That's unusable in any client-facing interface. This workflow fixes it.

## What it does

Uses an OpenAI Assistant with a vector store (file retrieval enabled), fetches the complete thread via API to get all citation metadata, maps each citation ID back to the actual source filename, then formats the final output with clean inline `_(filename)_` references. Optional Markdown-to-HTML conversion at the end.

## Tools Used
- **Orchestration:** n8n
- **AI:** OpenAI Assistants API (v2)
- **Memory:** Window Buffer Memory
- **Logic:** HTTP requests to thread API + JS code node

## Flow
```
Chat Trigger
  → OpenAI Assistant (with Vector Store)
  → Fetch complete thread content (HTTP)
  → Split message iterations
  → Split citations from each message
  → Fetch filename for each file ID
  → Aggregate file + citation pairs
  → Format output (replace markers with filenames)
  → Optional: Markdown to HTML
```

## Why I built this
Any RAG system I deploy for clients needs proper source attribution. OpenAI's default citation format was the blocker — this workflow makes it clean and readable.
