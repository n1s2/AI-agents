# FLOOWBOX - Sales Pipeline Status Reporter

Every Friday at 5 PM, this workflow generates a full pipeline briefing so Monday morning has a clear action plan — no scrambling to remember where deals stand.

## What it does

Fetches all active deals from Airtable (excluding Closed Lost). Calculates pipeline value by stage — Prospecting, Proposal, Negotiation, etc. GPT-4o writes a focused weekly summary: what's hot, what needs attention, and one specific action for Monday. Saves history to Google Sheets for trend tracking. Emails the report at end of week.

## Tools Used
- **Orchestration:** n8n
- **Data:** Airtable (Deals table)
- **AI:** OpenAI GPT-4o
- **Storage:** Google Sheets (weekly history)
- **Email:** SMTP
- **Schedule:** Every Friday at 5 PM

## Flow
```
Friday 5 PM
  → Fetch all active deals from Airtable
  → Aggregate and calculate:
      - Total pipeline value
      - Count and value by stage
      - Hot deals (Proposal + Negotiation)
  → GPT-4o writes pipeline summary
  → Save to Google Sheets
  → Email weekly report
```

## Sample report
```
Pipeline is healthy at ₹8.4L across 12 active deals.
3 deals in Negotiation stage represent ₹3.1L — close
rate needed: 2/3 to hit monthly target.

Hot deal to focus: Acme Corp (₹1.8L, deadline Mar 31).
Action Monday: Send revised proposal to TechStartup Ltd.
```

## Why I built this

Without a weekly review, deals stall silently. This forces a Friday review automatically — even if you don't read it carefully, having the pipeline value in your inbox every Friday keeps you honest about where the business actually stands.

## Airtable fields needed

| Field | Type |
|---|---|
| Stage | Select (Prospecting/Proposal/Negotiation/etc.) |
| Value | Number (in ₹) |
| Status | Select (Active/Closed Won/Closed Lost) |
| Company | Text |
| Contact | Text |

## Setup
1. Airtable base with Deals table (schema above)
2. OpenAI API key
3. Google Sheets ID
4. SMTP credentials
