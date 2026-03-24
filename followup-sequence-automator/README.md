# FLOOWBOX - Follow-up Sequence Automator

Most deals don't close on the first email. This workflow runs a 3-touch follow-up sequence automatically — each email written fresh by GPT-4o, not templated.

## What it does

Runs every morning at 9 AM. Checks Airtable for leads who were contacted but haven't replied, and are due for a follow-up based on timing. Routes each lead to the right email based on how many follow-ups they've received. GPT-4o writes a different angle for each touch — not the same email resent. After 3 follow-ups, status updates to "Sequence Complete" and the lead stops receiving emails.

## Tools Used
- **Orchestration:** n8n
- **AI:** OpenAI GPT-4o (3 different email styles)
- **CRM:** Airtable
- **Email:** SMTP
- **Logic:** Switch node (routes by follow-up count)

## Sequence

| Touch | Timing | Angle |
|---|---|---|
| Follow-up 1 | Day 3 | Reference original email + new value point |
| Follow-up 2 | Day 7 | Completely different angle + case study |
| Breakup | Day 14 | Close the loop, leave door open |

## Flow

```
9 AM daily
  → Fetch leads: status=Contacted, follow_ups < 3, timing due
  → Switch by follow-up count (0/1/2)
  → GPT-4o writes appropriate email
  → Send email
  → Update: follow_up_count + 1, last_follow_up_date
  → If count = 3: status = Sequence Complete
```

## Why I built this

The difference between 5% and 15% reply rates is almost entirely follow-up consistency. Most people give up after one email. This runs the full sequence automatically — and because each email is written fresh with a different angle, it doesn't feel like a drip campaign.

## Pairs with

1. [LinkedIn Lead Scraper](../linkedin-lead-scraper-enrichment/) — fills Airtable with qualified leads
2. [Cold Email Personalization Agent](../cold-email-personalization-agent/) — sends the first email
3. This workflow — handles all follow-ups automatically

Together these three form a complete outbound pipeline.

## Airtable fields needed

| Field | Type | Notes |
|---|---|---|
| Status | Select | New Lead / Contacted / Sequence Complete |
| Follow Up Count | Number | Starts at 0 |
| Contacted Date | Date | Set by cold email workflow |
| Next Follow Up Days | Number | 3 for FU1, 7 for FU2, 14 for breakup |
| Last Follow Up Date | Date | Updated each run |
| Email Subject Sent | Text | Original subject line |

## Setup

1. Airtable base + Leads table (schema above)
2. OpenAI API key
3. SMTP credentials
