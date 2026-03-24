# FLOOWBOX - LinkedIn Lead Scraper and Enrichment Agent

Manual LinkedIn prospecting — search, click, copy, paste — was taking 3-4 hours per week. This does the same job in under 5 minutes and only sends qualified leads to the CRM.

## What it does

Sets search parameters (job title, industry, location). Apify scrapes matching LinkedIn profiles. GPT-4o scores each profile against the Ideal Customer Profile — gives an ICP match score out of 10, identifies likely pain points, and generates a personalized one-line outreach hook for each person. Only leads scoring 6 or above make it to Airtable. Low-quality profiles are filtered out automatically.

## Tools Used
- **Orchestration:** n8n
- **Scraping:** Apify (LinkedIn Profile Scraper actor)
- **AI Enrichment:** OpenAI GPT-4o
- **CRM:** Airtable
- **Logic:** Score-based filtering (6+/10 threshold)

## Flow

```
Set search params (title, industry, location)
  → Apify scrapes LinkedIn profiles
  → Wait 30s for run to complete
  → Fetch results from Apify dataset
  → Split into individual profiles
  → GPT-4o enriches each:
      - ICP match score (1-10)
      - Pain points
      - Personalized outreach angle
  → Filter: score >= 6 only
  → Save qualified leads to Airtable
```

## What GPT-4o returns per profile

```json
{
  "icp_match": true,
  "icp_score": 8,
  "pain_points": ["scaling ops without hiring", "manual reporting overhead"],
  "outreach_angle": "Noticed you scaled from 10 to 50 clients — most founders at that stage are drowning in manual ops",
  "best_time_to_contact": "morning"
}
```

## Why I built this

FLOOWBOX's target clients are founders and ops leads at growing companies. Finding them manually on LinkedIn was the biggest time sink in the sales process. This generates a qualified, enriched lead list with personalized hooks — ready to drop straight into a cold email.

## Customization

Change `job_title`, `industry`, and `location` in the Set Search Params node. Adjust the ICP score threshold in the Filter node (default: 6/10).

## Setup

1. Apify account + API key
2. OpenAI API key
3. Airtable base ID and Leads table
4. LinkedIn session cookies in Apify actor config
