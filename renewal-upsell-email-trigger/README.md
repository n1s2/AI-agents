# FLOOWBOX - Renewal Upsell Email Trigger

Renewal time is the best moment to expand revenue — the customer is already deciding whether to stay. This workflow identifies the right upsell angle for each customer and sends a personalized email automatically.

## What it does

Runs daily. Finds all active customers with renewals within the next 60 days who haven't received an upsell email yet. Fetches their usage data from Google Sheets. GPT-4o analyzes the combination — are they hitting plan limits, barely using the product, have multiple team members, or showing strong ROI? Based on this, it picks the right upsell type and writes a personalized 150-word email. High-confidence opportunities send automatically. Edge cases get a Slack notification for manual review.

## Tools Used
- **Orchestration:** n8n
- **CRM:** Airtable
- **Usage Data:** Google Sheets
- **AI:** OpenAI GPT-4o (opportunity analysis + email writing)
- **Email:** SMTP
- **Alerts:** Slack (#revenue)
- **Schedule:** Daily 8 AM

## Upsell types GPT-4o identifies

| Signal | Upsell Type | Offer |
|---|---|---|
| Hitting plan limits | Tier upgrade | "You're outgrowing your current plan" |
| Low feature usage | Training package | "Free onboarding session to unlock value" |
| Multiple team logins | Team/seats expansion | "Add your whole team at a discount" |
| High activity + ROI | Annual upgrade | "Lock in 20% savings with annual billing" |

## Flow

```
Daily 8 AM
  → Find customers: renewal within 60 days + no upsell sent
  → Fetch usage signals (Google Sheets)
  → GPT-4o identifies upsell type + writes email
  → IF send_email = true: send + mark sent in Airtable
  → Log opportunity to Slack #revenue
```

## Why I built this

Revenue expansion from existing customers costs 5x less than acquiring new ones. A SaaS client was only doing renewal conversations reactively — when customers called. This makes the first move 60 days early, with a relevant offer based on actual usage data.

## Setup

1. Airtable: Customers table with Renewal Date, Upsell Email Sent, Plan fields
2. Google Sheets: Customer usage log
3. OpenAI API key
4. SMTP credentials
5. Slack Bot Token + #revenue channel
