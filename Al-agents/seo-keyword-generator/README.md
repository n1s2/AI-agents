# FLOOWBOX - AI SEO Seed Keyword Generator

The first step of every client SEO engagement is defining seed keywords. This used to take an hour of manual research. Now it takes 2 minutes.

## What it does

You define your Ideal Customer Profile (ICP) — product, pain points, goals, current solutions they use, expertise level. The AI Agent analyzes this and generates 15-20 targeted seed keywords covering all search intent types: informational, navigational, commercial, and transactional. Output is a clean array ready to plug into your keyword research tool.

## Tools Used
- **Orchestration:** n8n
- **LLM:** Anthropic Claude Sonnet (via n8n Anthropic node)
- **Output:** Any database / Google Sheets / Airtable (configurable)

## Flow
```
Manual Trigger
  → Set ICP fields (product, pain points, goals, current solutions, expertise)
  → Aggregate into single object
  → AI Agent (Claude Sonnet)
      - Thinks through keyword ideas across funnel stages
      - Considers industry jargon, tools, integrations
      - Mixes head terms with more specific phrases
  → Split output array
  → Save to database / GSheet / Airtable
```

## ICP fields to fill in
```
product          → What you're selling
pain points      → What problems your customer has
goals            → What outcomes they want
current solutions → How they solve it today (your competition)
expertise level  → Beginner / intermediate / expert
```

## Example output (B2B SaaS client)
```
['b2b lead generation', 'sales automation software', 'crm integration tools',
'outbound email sequences', 'pipeline management', 'saas sales funnel', ...]
```

## Why I built this

Every FLOOWBOX client needs an SEO foundation before we build content workflows. Manually brainstorming seed keywords is slow and often misses angles. Claude's analysis of the ICP consistently surfaces keywords I wouldn't have thought of immediately — especially around the "current solutions" angle.

## Cost
~$0.02–0.05 per run using Claude Sonnet 3.5. Essentially free at any reasonable usage volume.
