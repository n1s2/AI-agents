# FLOOWBOX - LinkedIn Skills Trend Analyzer

The skills that get you hired change faster than most people update their resume. This workflow monitors what is rising and falling in the job market every month and generates a personalized learning plan based on skills you already have.

## What it does

Runs on the 1st of every month. Three Perplexity searches run in parallel — identifying which skills are growing in job posting frequency, which are declining and being replaced, and which certifications are gaining traction. GPT-4o cross-references the market trends against your current skills list to identify exactly which rising skills you are missing, which of your existing skills are becoming less relevant, and a prioritized 6-month learning plan.

## Tools Used
- **Orchestration:** n8n
- **Trend Research (x3):** Perplexity AI Sonar (parallel)
- **Gap Analysis:** OpenAI GPT-4o
- **Storage:** Notion
- **Report:** Slack
- **Schedule:** Monthly on 1st

## Three research streams

| Stream | What it tracks |
|---|---|
| Rising Skills | Skills with growing job posting frequency vs 3 months ago |
| Declining Skills | Skills being replaced or mentioned less frequently |
| Certifications | New and established certifications gaining employer value |

## Personalized gap analysis output

```json
{
  "skills_i_have_that_are_rising": ["Python", "LangChain", "n8n"],
  "skills_i_have_that_are_declining": [],
  "priority_skills_to_learn": [
    {
      "skill": "LLM fine-tuning (LoRA/QLoRA)",
      "reason": "Mentioned in 43% more job postings vs 3 months ago",
      "resource": "Hugging Face PEFT course (free)",
      "time_needed": "2-3 weeks"
    }
  ],
  "certifications_worth_pursuing": [
    {"name": "AWS Certified ML Specialty", "provider": "AWS", "time": "3 months", "cost": "$300"}
  ],
  "6_month_learning_plan": [
    "Month 1-2: LLM fine-tuning fundamentals",
    "Month 3: AWS ML certification prep",
    "Month 4-5: Multi-agent orchestration patterns",
    "Month 6: Deploy personal project using new skills"
  ]
}
```

## Why I built this

Skills decay faster in AI than any other field. GPT-2 skills are not the same as GPT-4o skills. A monthly skills audit keeps the learning direction aligned with where the market is actually going rather than where it was when you last checked.

## Setup

1. Update `current_skills` in Set Skills Config with your actual skills
2. Perplexity API key
3. OpenAI API key
4. Notion integration + DB ID
5. Slack Bot Token + #learning channel
