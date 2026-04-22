# FLOOWBOX - Autonomous Task Planner Agent

The capstone workflow of the 30-day series. Give this agent any goal — it plans, executes, and reports back. This implements the Plan-Execute-Evaluate loop at the heart of modern autonomous agent architectures.

## What it does

State a goal in plain language. A Planner agent decomposes it into a step-by-step execution plan, specifying which tool to use at each step and what success looks like. An Executor agent works through the plan using available tools (web search, file operations, APIs). An Evaluator agent reviews the execution output, measures completion percentage, and identifies what was and wasn't achieved.

## Tools Used
- **Orchestration:** n8n
- **Planner:** OpenAI GPT-4o (goal decomposition)
- **Executor:** OpenAI GPT-4o (plan execution + SerpAPI)
- **Evaluator:** OpenAI GPT-4o (completion assessment)
- **Search:** SerpAPI
- **Storage:** Notion
- **Trigger:** Chat interface

## Three-agent architecture

```
Goal
  → Planner: decompose into steps + tools + expected outputs
  → Executor: work through each step methodically
      → Uses: web search, reasoning, tool calls
  → Evaluator: assess completion percentage
      → Reports: what was achieved, what wasn't, next actions
  → Save plan + results to Notion
```

## Example goals

- "Research the top 5 competitors for FLOOWBOX and summarize their pricing"
- "Find the 3 most relevant AI conferences in India in the next 6 months"
- "Draft a cold email for reaching out to logistics companies about automation"
- "Find me 5 GitHub repositories trending in multi-agent systems this week"

## Plan structure per step

```json
{
  "step": 2,
  "action": "Search for pricing pages of each identified competitor",
  "tool": "web search",
  "input": "Zapier pricing 2026",
  "expected_output": "Pricing tiers and amounts",
  "depends_on": [1]
}
```

## Why this is the capstone

This implements the **ReAct pattern** (Reasoning + Acting) — one of the most influential papers in the LLM agent literature (Yao et al., 2022). The three-stage architecture (Plan → Execute → Evaluate) directly maps to the core architecture used in AutoGPT, BabyAGI, and the first generation of autonomous agents.

Building this in n8n demonstrates:
- Understanding of agent architecture fundamentals
- Goal decomposition and task planning
- Tool integration in agentic systems
- Completion verification and self-evaluation

These are core topics at every AI research lab — MBZUAI, Stanford HAI, MIT CSAIL, and CMU LTI all have active research in autonomous agent systems.

## The 30-day journey in one workflow

This agent can actually be given the meta-goal: *"Summarize the FLOOWBOX AI automation workflow portfolio and identify the 5 most impressive workflows for MS in AI admissions."* And it will plan, search, analyze, and report back.

## Setup

1. OpenAI API key
2. SerpAPI key
3. Notion integration + Task Log DB
