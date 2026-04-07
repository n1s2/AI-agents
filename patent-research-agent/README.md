# FLOOWBOX - Patent Research Agent

Patent landscape analysis usually requires expensive legal software or days of manual Google Patents searches. This agent does it in under 3 minutes using three parallel research streams.

## What it does

Send a technology area and optionally a company name. Three Perplexity AI searches run simultaneously — one finding recent relevant patents, one mapping the prior art landscape and major holders, one checking for active litigation and disputes. GPT-4o synthesizes all three into a structured patent intelligence report: landscape density (crowded/moderate/open), key patent holders, white spaces for new filings, litigation risk, and competitive insights.

## Tools Used
- **Orchestration:** n8n
- **Research (x3):** Perplexity AI Sonar (parallel)
- **Synthesis:** OpenAI GPT-4o
- **Storage:** Notion
- **Trigger:** Webhook

## Three parallel searches

| Stream | What it finds |
|---|---|
| Recent Patents | Top 5 most relevant recent filings with patent numbers |
| Prior Art | Foundational patents, major holders, contested vs open areas |
| Litigation | Active disputes, lawsuits, licensing controversies |

## Report output

```json
{
  "patent_landscape": "crowded",
  "key_patent_holders": ["Google", "Microsoft", "Meta", "OpenAI"],
  "white_spaces": ["Edge inference optimization", "Multilingual fine-tuning efficiency"],
  "litigation_risk": "medium",
  "active_disputes": ["Google vs Anthropic (pending)"],
  "filing_recommendations": ["Focus on novel training data curation methods — less contested"],
  "summary": "The transformer attention mechanism space is heavily crowded with 200+ patents filed in 2024..."
}
```

## Use cases

- **IP strategy**: Identify where to file new patents
- **Freedom to operate (FTO)**: Check if your technology infringes existing patents
- **Competitive intelligence**: Map competitor patent activity
- **Due diligence**: Pre-acquisition patent landscape assessment
- **Research**: Understanding IP landscape before publishing

## Why I built this

A FLOOWBOX client in the AI tools space needed to know whether their novel training approach was patentable and what competitive IP existed. Traditional patent searches took 2-3 weeks. This does the landscape analysis in minutes — enough to have an informed initial conversation with IP counsel.

## Setup

1. Perplexity API key
2. OpenAI API key
3. Notion integration + DB ID
