# FLOOWBOX - Grant Opportunity Finder Agent

Funding opportunities have deadlines that expire quietly. This workflow monitors scholarships, government grants, and industry fellowships every week — specifically for Indian AI/ML students applying to top programs — so nothing slips through.

## What it does

Every Monday, three parallel Perplexity searches run: one for global scholarships at target programs (MBZUAI, Stanford, MIT, Purdue), one for Indian government funding (ICCR, MHRD, DST), and one for industry fellowships (Google, Microsoft, Anthropic, OpenAI, Meta). GPT-4o compiles and prioritizes all findings by eligibility match and deadline urgency, calculates total potential funding available, and identifies the highest-priority action for the week.

## Tools Used
- **Orchestration:** n8n
- **Research (x3):** Perplexity AI Sonar (parallel)
- **Prioritization:** OpenAI GPT-4o
- **Storage:** Notion
- **Alert:** Slack
- **Schedule:** Weekly Monday 8 AM

## Why this workflow exists

Building an MS in AI application portfolio is only half the battle — funding is the other half. MBZUAI is fully funded, but other target programs require external fellowships. Missing a deadline by one week means waiting another year. This makes funding research systematic instead of ad-hoc.

## Funding sources tracked

| Category | Sources |
|---|---|
| Program-specific | MBZUAI fully funded, Stanford fellowships, MIT EECS fellowships |
| Indian Government | ICCR, MHRD overseas scholarships, DST-SERB fellowships |
| Industry | Google Fellowship, Microsoft Research, Anthropic, Meta AI, Apple |

## Example output

```json
{
  "urgent_deadlines": [
    {"name": "Google PhD Fellowship", "amount": "$50,000/year", "deadline": "Apr 30 2026", "eligibility_match": "high"}
  ],
  "upcoming_opportunities": [
    {"name": "MBZUAI MS Scholarship", "amount": "Fully funded + stipend", "opens": "Aug 2026", "notes": "Auto-considered on admission"}
  ],
  "highest_priority_action": "Submit Google Fellowship application before Apr 30 — eligibility matches perfectly",
  "total_potential_funding": "$150,000+ across 4 open opportunities"
}
```

## Customization

Update the `Set Research Profile` node with your specific research area, target programs, and applicant profile. The workflow personalizes all searches based on this config.

## Setup

1. Perplexity API key
2. OpenAI API key
3. Notion integration + DB ID
4. Slack Bot Token + #ms-applications channel
