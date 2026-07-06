# competitor-intel-monitor

Keeping up with competitors manually means either checking too infrequently or spending hours every week. This runs every Monday morning, searches for recent news on each competitor, and delivers a digest of what's actually new — product updates, pricing changes, funding rounds, leadership moves — with a significance rating and implication for your company. Nothing from this week, nothing in the digest.

---

## What it does

**Weekly (Monday 7am):** Loads competitor list from Google Sheets, fires a Tavily search per competitor, Claude analyzes results for competitively relevant updates (categorized, significance-rated, with implication for you), assembles digest, emails to recipient list.

**On-demand (webhook):** Same flow, triggered immediately. Useful for ad-hoc scans before a board meeting or sales call.

For each competitor: notable updates with category (product/pricing/funding/partnership/leadership/marketing/legal), significance (high/medium/low), and what it means for your company. Overall threat direction: increasing/stable/decreasing. One-line summary for quick scanning.

---

## Stack

n8n (weekly scheduler + webhook), Google Sheets, Tavily API, Anthropic Claude (claude-sonnet-4-20250514), SMTP.

---

## Setup

**Sheet "Competitors"** columns: `competitor_name | domain | focus_areas | your_company | recipient_emails | active`

Example row: `Asana | asana.com | pricing,product,funding | Flowdesk | ceo@flowdesk.com,product@flowdesk.com | true`

**Env vars:** `COMPETITOR_SHEET_ID`, `TAVILY_API_KEY`, `FROM_EMAIL`

---

## On-demand webhook

```bash
curl -X POST https://your-n8n.com/webhook/run-competitor-intel \
  -H "Content-Type: application/json" \
  -d '{
    "your_company": "Flowdesk",
    "recipient_emails": ["team@flowdesk.com"],
    "competitors": [
      {"name": "Asana", "domain": "asana.com", "focus_areas": ["pricing", "product", "AI features"]},
      {"name": "Monday.com", "focus_areas": ["enterprise", "pricing", "partnerships"]}
    ]
  }'
```

Pass `competitors` to override the Sheet — useful for one-off scans on specific companies.

---

## Limitations

- Searches are Tavily web results — real-time indexed content, not proprietary data sources. Competitor pricing page changes or gated announcements may not appear.
- Low-significance weeks still send a digest. If a week has nothing new, summaries will reflect that.

---

## License

MIT.
