# FLOOWBOX - AI Agent with Long-Term Memory (Mem0 Pattern)

Standard LLM agents forget everything between sessions. This agent remembers — user preferences, goals, past context, important decisions — and uses that knowledge naturally in every future conversation, without the user having to repeat themselves.

## What it does

Every conversation starts by retrieving relevant memories about the user from a Qdrant vector database. These memories are injected into the agent's system prompt as context. The agent responds naturally, using the memory without explicitly announcing it. After each exchange, a second GPT-4o pass extracts new facts worth remembering. Those facts are embedded and stored back into Qdrant for future sessions.

## Tools Used
- **Orchestration:** n8n
- **Memory Store:** Qdrant (vector database, local)
- **Embeddings:** OpenAI text-embedding-3-small
- **Agent:** OpenAI GPT-4o (conversation)
- **Memory Extraction:** OpenAI GPT-4o (second pass)
- **Trigger:** Chat interface

## Memory lifecycle

```
Session starts
  → Retrieve user memories from Qdrant (filtered by user_id)
  → Format as context string
  → Inject into agent system prompt
  → User sends message
  → Agent responds using context naturally
  → Extract new facts from exchange
  → IF worth saving:
      → Embed new facts
      → Upsert to Qdrant
Session ends — memories persist
```

## What gets remembered

The memory extractor only saves substantive facts:
- User's goals and preferences
- Technical stack they're working with
- Decisions they've made
- Context about their business
- Things they've asked to remember

It skips: pleasantries, one-time requests, transient information.

## Example memory in action

**Session 1:**
User: "I'm building an automation system for my Mumbai-based logistics company"

**Session 2 (different conversation):**
Agent: "Based on your logistics company in Mumbai, this workflow pattern would work well with the high-volume order processing you're likely dealing with..."

The user never said "logistics" in session 2 — the agent remembered.

## Why this is research-relevant

Long-term memory for LLM agents is an active research area. Key papers include Mem0 (official implementation), MemGPT (memory management with context limits), and several NeurIPS 2024 papers on episodic memory in neural networks. This n8n implementation demonstrates the core pattern in a production-deployable system.

The two-pass architecture — agent responds first, then a separate pass extracts what to remember — avoids contaminating the user experience with memory management overhead.

## Qdrant collection setup

```bash
curl -X PUT http://localhost:6333/collections/user_memories \
  -H 'Content-Type: application/json' \
  -d '{"vectors": {"size": 1536, "distance": "Cosine"}}'
```

## Setup

1. Qdrant running locally (or Qdrant cloud)
2. OpenAI API key (embeddings + GPT-4o × 2)
