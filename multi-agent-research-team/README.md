# FLOOWBOX - Multi-Agent Research Team

A single LLM does decent research. Multiple specialized agents working together — each with a different role — produce research that is more thorough, more balanced, and more reliable. This workflow implements a full multi-agent research pipeline.

## What it does

A chat trigger accepts any research question. A Planner agent breaks it into 3-4 specific sub-tasks. Three specialist Researcher agents work the sub-tasks in parallel — one focused on general findings, one on data and statistics, one on expert opinions and case studies. A Critic agent reviews all three outputs and flags gaps, contradictions, and unsupported claims. Finally a Synthesis agent combines everything into a structured Markdown report, incorporating the critic's feedback. Report saves to Notion.

## Tools Used
- **Orchestration:** n8n
- **Agent Framework:** n8n LangChain agents (6 agents total)
- **LLM:** OpenAI GPT-4o (all agents)
- **Web Search:** SerpAPI (all researcher agents)
- **Storage:** Notion
- **Trigger:** Chat interface

## Agent Roles

| Agent | Role | Specialization |
|---|---|---|
| Planner | Breaks query into sub-tasks | Task decomposition |
| Researcher A | General web research | Broad coverage |
| Researcher B | Data and statistics | Quantitative evidence |
| Researcher C | Expert opinions | Case studies and examples |
| Critic | Reviews all findings | Gap and contradiction detection |
| Synthesizer | Final report writer | Coherent narrative |

## Flow

```
Chat question
  → Planner: decompose into 3 sub-tasks
  → Parallel research (A + B + C simultaneously)
  → Critic: review all three outputs
  → Synthesizer: write final report
  → Save to Notion
```

## Why this beats a single agent

Single agent research suffers from: shallow coverage, no self-correction, no specialization. The Critic agent alone improves output quality significantly — it forces claims to be substantiated and flags when the research only presents one side.

## Why I built this

This workflow demonstrates multi-agent orchestration — one of the core research areas at MBZUAI and Stanford AI labs. I built it to understand agent coordination patterns in practice, not just in theory. The Critic-Synthesizer loop is directly analogous to Constitutional AI and RLHF feedback patterns.

## Setup

1. OpenAI API key
2. SerpAPI key
3. Notion integration + Research DB ID
