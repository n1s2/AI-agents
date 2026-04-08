# FLOOWBOX - Job Market Intelligence Tracker

The AI job market changes week to week — companies announce hiring sprees, others freeze. This workflow monitors job demand, salary signals, and hiring patterns every Monday so you always know where the opportunities are.

## What it does

Every Monday at 8 AM, three Perplexity searches run in parallel tracking the job market for AI and automation roles. First stream finds current job posting volumes and which companies are hiring most. Second stream identifies companies announcing layoffs vs hiring surges. Third stream pulls current salary ranges from Glassdoor, LinkedIn Salary, and Levels.fyi. GPT-4o synthesizes all three into a weekly market intelligence brief with a "market temperature" score and specific action items.

## Tools Used
- **Orchestration:** n8n
- **Market Research (x3):** Perplexity AI Sonar (parallel)
- **Synthesis:** OpenAI GPT-4o
- **Trend Logging:** Google Sheets
- **Brief:** Slack
- **Schedule:** Weekly Monday 8 AM

## Flow

```
Monday 8 AM
  → Parallel Perplexity searches:
      LinkedIn job trends + top hiring companies
      Company hiring/layoff signals
      Current salary ranges
  → GPT-4o: compile market intelligence
  → Log to Google Sheets (weekly trend)
  → Post Slack brief
```

## Market intelligence output

```json
{
  "market_temperature": "hot",
  "top_hiring_companies": [
    {"company": "Razorpay", "roles": ["ML Engineer", "AI Platform"], "urgency": "high"}
  ],
  "most_demanded_skills": [
    {"skill": "LangChain", "frequency": "very high"},
    {"skill": "n8n", "frequency": "high"},
    {"skill": "RAG architecture", "frequency": "high"}
  ],
  "salary_ranges": [
    {"role": "AI Engineer", "min": "₹12L", "max": "₹28L", "median": "₹18L"}
  ],
  "action_items": [
    "Apply to Razorpay — 3 AI roles posted this week",
    "Add RAG experience to resume — mentioned in 67% of JDs"
  ]
}
```

## Why I built this

Building a career in AI requires knowing where the market is moving, not where it was 3 months ago. This gives a weekly snapshot — which companies are actually hiring, what skills they are paying for, and where salaries are headed. I use this to stay current even while working on FLOOWBOX.

## Setup

1. Perplexity API key
2. OpenAI API key
3. Google Sheets ID
4. Slack Bot Token + #job-market channel
5. Update roles and location in Set Market Config node
