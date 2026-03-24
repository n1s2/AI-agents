# FLOOWBOX - Cold Email Personalization Agent

Generic cold emails get ignored. This workflow researches every prospect in real time and writes a unique email for each one — at scale.

## What it does

Every morning at 8 AM, fetches up to 10 new leads from Airtable with status "New Lead". For each lead, Perplexity AI searches for recent company news — funding rounds, product launches, hiring activity, anything from the last month. GPT-4o then writes a personalized cold email using that fresh context plus the outreach angle from the LinkedIn scraper. Email sends, lead status updates to "Contacted".

## Tools Used
- **Orchestration:** n8n
- **Real-time Research:** Perplexity AI Sonar (recent company news)
- **Email Writing:** OpenAI GPT-4o
- **CRM:** Airtable
- **Email:** SMTP
- **Schedule:** Daily at 8 AM, 10 leads per run

## Flow

```
8 AM daily trigger
  → Fetch "New Lead" records from Airtable (max 10)
  → For each lead:
      → Perplexity: find recent news about their company
      → GPT-4o: write personalized email (subject + body)
      → Send email
      → Update Airtable: status = Contacted
```

## Why this works better than templates

Each email references something real and recent — "Saw you just raised a seed round" or "Noticed you're hiring 3 engineers right now." That specificity is what gets replies. GPT-4o keeps it under 120 words and peer-to-peer in tone.

## Why I built this

FLOOWBOX's outbound was inconsistent — some weeks I'd send 20 emails, some weeks zero. Automating it at 10/day means the pipeline never dries up, and every email is genuinely personalized, not mail-merged.

## Pairs with

Use after the [LinkedIn Lead Scraper](../linkedin-lead-scraper-enrichment/) — that workflow fills Airtable with qualified leads, this one works through them every morning.

## Setup

1. Airtable base with Leads table (from LinkedIn scraper)
2. Perplexity API key (Header Auth: `Authorization: Bearer YOUR_KEY`)
3. OpenAI API key
4. SMTP credentials
5. Set sending limit in Limit node (default: 10/day)
