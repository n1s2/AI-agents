# FLOOWBOX - Research Funding and Collaboration Tracker

Getting into MBZUAI or Stanford isn't just about grades and test scores — it's about research fit, professor connections, and demonstrated interest. This workflow systematically identifies which professors are actively taking students, which open-source projects to contribute to, and what internships are available right now.

## What it does

Runs weekly. Three parallel Perplexity searches: one finding active research groups at target labs (MBZUAI, Stanford HAI, MIT CSAIL, CMU LTI) that are currently working in the research focus area, one finding relevant open-source AI projects seeking contributors, one finding research internship and visiting researcher opportunities. GPT-4o generates a personalized action plan — which professors to email with what contact strategy, which repos to contribute to, and specific actions for the current week and 30-day horizon.

## Tools Used
- **Orchestration:** n8n
- **Research (x3):** Perplexity AI Sonar (parallel)
- **Action Planning:** OpenAI GPT-4o
- **Storage:** Notion
- **Report:** Slack
- **Schedule:** Weekly Monday 7 AM

## Why cold emailing professors works

Top MS programs value students who demonstrate genuine research interest. A well-written cold email referencing a professor's recent paper — sent months before the application deadline — creates a relationship. Many admitted students report that a positive professor reply ("I'd be happy to work with you") significantly influenced their admission decision.

## Action plan output

```json
{
  "target_professors": [
    {
      "name": "Dr. Salman Khan",
      "lab": "MBZUAI Vision Lab",
      "research_match": "high",
      "recent_work": "Published multi-agent visual reasoning paper Feb 2026",
      "contact_strategy": "Reference the Feb 2026 paper, mention your RAG verification agent as related work"
    }
  ],
  "open_source_contributions": [
    {
      "project": "LangGraph",
      "why_relevant": "Directly implements the multi-agent patterns in your portfolio",
      "entry_point": "Good first issues tagged — documentation and example notebooks"
    }
  ],
  "this_week_actions": [
    "Send cold email to Dr. Salman Khan referencing Feb 2026 paper",
    "Open first PR on LangGraph examples repo",
    "Register for MBZUAI open day webinar (link in Notion)"
  ]
}
```

## Personal note

This is the workflow I use for my own Fall 2027 MS applications. The research profile is set to my actual targets — MBZUAI (priority), Stanford, MIT. Every week it surfaces new connection opportunities I would otherwise miss.

## Setup

1. Perplexity API key
2. OpenAI API key
3. Notion integration + Research Network DB
4. Slack Bot Token + #ms-applications channel
5. Update research profile with your specific focus area and targets
