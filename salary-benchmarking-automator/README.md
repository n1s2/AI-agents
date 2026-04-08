# FLOOWBOX - Salary Benchmarking Automator

Walking into a salary negotiation without data is guesswork. This workflow researches current market rates across India and globally, breaks them down by company type, and identifies exactly which skills command the highest premiums.

## What it does

Send a role title, years of experience, and location. Three parallel Perplexity searches run — one finding current India salaries from AmbitionBox, Glassdoor, LinkedIn, and Naukri broken down by startup/mid-size/MNC; one finding global salary data from Levels.fyi and LinkedIn across US, UK, UAE, Singapore, and Canada; one researching skill-specific premiums for the listed technologies. GPT-4o compiles everything into a benchmarking report with negotiation tips and red flags to watch for in offers.

## Tools Used
- **Orchestration:** n8n
- **Research (x3):** Perplexity AI Sonar (parallel)
- **Analysis:** OpenAI GPT-4o
- **Database:** Google Sheets
- **Trigger:** Webhook

## Three research streams

| Stream | Data Sources | Coverage |
|---|---|---|
| India Salaries | AmbitionBox, Glassdoor IN, LinkedIn, Naukri | Startup / Mid-size / MNC breakdown |
| Global Salaries | Levels.fyi, LinkedIn Salary, Glassdoor | US, UK, UAE, Singapore, Canada |
| Skill Premiums | LinkedIn, job posting analysis | % salary increase per skill |

## Benchmark output

```json
{
  "india_benchmark": {
    "startup": {"min": "₹10L", "max": "₹20L", "median": "₹14L"},
    "mid_size": {"min": "₹14L", "max": "₹26L", "median": "₹18L"},
    "mnc": {"min": "₹18L", "max": "₹35L", "median": "₹24L"}
  },
  "global_benchmark": [
    {"country": "USA", "median_usd": "$145,000", "total_comp": "$180,000 with equity"},
    {"country": "UAE", "median_usd": "$85,000", "total_comp": "Tax-free"}
  ],
  "skill_premiums": [
    {"skill": "LLM fine-tuning", "premium_percent": 25},
    {"skill": "RAG architecture", "premium_percent": 18}
  ],
  "negotiation_tips": ["Counter with total comp not just base", "Equity cliff periods matter more than headline number"]
}
```

## Why I built this

Evaluating job offers without benchmarks means accepting whatever sounds reasonable. This gives hard data before any negotiation conversation — making it fact-based instead of gut-feel.

## Webhook payload

```json
{
  "role": "AI Engineer",
  "years": 2,
  "location": "Bangalore, India",
  "skills": "Python, LangChain, RAG, n8n",
  "company_size": "startup"
}
```

## Setup

1. Perplexity API key
2. OpenAI API key
3. Google Sheets ID for salary database
