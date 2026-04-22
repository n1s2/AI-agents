# FLOOWBOX - Multi-Agent Debate System

Most AI systems give you one answer. This system gives you the strongest case for both sides of any question — then has an impartial judge synthesize the truth. Five specialized agents collaborate in a structured adversarial dialogue.

## What it does

Ask any question. A Proposer agent builds the strongest affirmative argument. An Opposer agent builds the strongest negative argument. Both run in parallel. Then each agent reads the other's opening and delivers a targeted rebuttal. Finally a Judge agent — with no prior stake in either position — evaluates the full debate, identifies what each side missed, and synthesizes the most intellectually honest answer to the original question.

## Tools Used
- **Orchestration:** n8n
- **Agent Framework:** n8n LangChain agents (5 agents)
- **LLM:** OpenAI GPT-4o (all agents)
- **Storage:** Notion (full debate transcript)
- **Trigger:** Chat interface

## Debate architecture

```
Question
  → Parallel Round 1:
      Agent A (Proposer): strongest affirmative case
      Agent B (Opposer): strongest negative case
  → Round 2 (each sees opponent's Round 1):
      Agent A Rebuttal: counter opponent's specific points
      Agent B Rebuttal: counter opponent's specific points
  → Judge: evaluate full debate → impartial verdict + synthesis
```

## Example question types

- "Should FLOOWBOX focus on Indian market first or go global?"
- "Is RAG better than fine-tuning for production LLM applications?"
- "Will autonomous AI agents replace SaaS tools by 2030?"
- "Is technical debt worse than slow development?"

## Why this is research-relevant

This implements several active research areas in AI:

**Adversarial collaboration** — structuring agent disagreement productively rather than converging on consensus. Recent papers on Constitutional AI and debate-based alignment use similar patterns.

**Role specialization in multi-agent systems** — agents perform better when given specific, constrained roles rather than general "answer this" prompts. This is a core finding in multi-agent LLM research.

**Impartial evaluation** — the Judge agent receiving a structured transcript and being asked for synthesis rather than continuation is directly analogous to reward model training in RLHF.

This workflow demonstrates practical understanding of the multi-agent research landscape — not just "use multiple LLM calls" but structuring agent interactions with intent.

## Why I built this

For FLOOWBOX clients making high-stakes decisions — pricing strategy, market entry, technology choices — one-sided AI advice is dangerous. This system forces both sides to be argued strongly before synthesizing. Better decisions come from adversarial thinking than from confirmation.

## Setup

1. OpenAI API key
2. Notion integration + DB ID
