# FLOOWBOX - HN Job Market Intelligence Scraper

Built this because manually reading through HN "Who is Hiring" threads every month was taking too long — and there was real signal buried in there for client research.

## What it does

Hits the HN Algolia API, finds the latest "Who is Hiring" thread, fetches every job listing comment, cleans the raw text, and runs it through OpenAI with a structured output parser to extract fields like company, role, location, tech stack, remote status, and salary range. Final output goes to Airtable.

## Tools Used
- **Orchestration:** n8n
- **Data:** HN Firebase API + Algolia search
- **Extraction:** OpenAI GPT + Structured Output Parser
- **Storage:** Airtable

## Flow
```
Manual/Schedule Trigger
  → Search HN Algolia for latest "Who is Hiring"
  → Get main post via HN API
  → Split out all children (individual job posts)
  → Fetch each post content
  → Clean raw text
  → LLM structured extraction
  → Write to Airtable
```

## Why I built this
FLOOWBOX clients often ask about hiring trends in their market. This gives structured data I can actually filter and analyze — not a wall of unformatted text.

## Commit date
First built: Jan 2026 | Updated: March 2026
