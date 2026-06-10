# newsletter-curator

Curating a weekly newsletter manually takes 2–3 hours. You search for recent content across your topics, decide what's worth including, write a short take on each piece, assemble it, and send it. Most of that is mechanical — the only part that requires human judgment is the editorial voice.

This automates the mechanical parts. Every Monday at 6am it searches Tavily across your configured topics, pulls the week's most relevant articles, and Claude selects the best ones, adds opinionated commentary, picks an editor's pick, and sends a formatted HTML newsletter to your inbox.

The editorial voice is yours to configure — set the tone, describe your audience, and the commentary adapts.

---

## What it does

**Weekly automated run (Monday 6am):**
- Loads configuration from Google Sheets (topics, recipient, tone, audience, item count)
- Searches Tavily for each topic (last 7 days)
- Logs raw articles to Google Sheets
- Aggregates and deduplicates all results
- Claude selects the best items, adds commentary, picks editor's pick, writes intro and closing
- Builds HTML newsletter
- Sends to configured recipient email

**Manual trigger (webhook):**
- POST to run immediately on demand

---

## Stack

- **n8n** — weekly scheduler + webhook
- **Tavily API** — article search (one search per topic)
- **Google Sheets** — config + raw article log
- **Anthropic Claude** (claude-sonnet-4-20250514) — curation + commentary
- **SMTP** — newsletter delivery

---

## Setup

### 1. Create the Google Sheet

Two tabs:

**Tab: Config** — two columns: `key` and `value`

Required config rows:
```
newsletter_name     | Weekly AI Digest
topics              | artificial intelligence, LLM tools, AI startups, machine learning research
recipient_email     | you@email.com
from_name           | AI Weekly
tone                | informative and slightly opinionated
items_per_issue     | 5
include_ai_commentary | true
audience_description | software engineers and product managers following AI developments
```

**Tab: Raw Articles** — columns:
```
fetched_at | topic | titles | urls
```

### 2. Environment variables

```
NEWSLETTER_SHEET_ID=your_google_sheet_id
TAVILY_API_KEY=tvly-your-key
FROM_EMAIL=newsletter@yourdomain.com
```

### 3. Credentials

- **Google Sheets OAuth2**
- **Anthropic API** (LangChain node)
- **SMTP**

### 4. Import and activate

Import `workflow.json`, activate. The newsletter sends every Monday at 6am. Test by triggering the weekly scheduler manually.

---

## Configuration

All settings live in the Config sheet. Change them there — no need to edit the workflow.

**`topics`** — comma-separated list of search queries. Each gets its own Tavily search. Be specific: `"venture capital Series B funding"` finds more relevant content than `"startup news"`. You can have 2–8 topics — more means more search API calls.

**`tone`** — describes the editorial voice. Examples:
- `"informative and neutral"` — objective, factual
- `"informative and slightly opinionated"` — has a point of view but balanced
- `"skeptical and critical"` — pushes back on hype
- `"enthusiastic and accessible"` — energetic, good for beginners
- `"dry and technical"` — for specialist audiences

**`items_per_issue`** — how many full items with commentary (default 5). Additional articles appear as quick links.

**`include_ai_commentary`** — set to `false` to get just summaries, no editorial voice.

**`audience_description`** — shapes the commentary tone and what Claude explains vs assumes. "ML researchers" gets different commentary than "non-technical founders interested in AI."

---

## The newsletter structure

**Editor intro** — 2–3 sentences on what's notable about this week's content. Claude is honest: if it was a slow news week, it says so.

**Selected items** (full items with commentary):
- Title (linked)
- Tag (one word category)
- Why selected (one sentence)
- Commentary (2–4 sentences of editorial perspective — not a headline summary but actual context or point of view)
- Editor's Pick star on the top item

**Quick links** — remaining notable items as one-liners without full commentary. Uses the rest of the high-quality articles that didn't make the main cut.

**Closing note** — 1–2 casual sentences from the editor.

---

## Sending to a list

The current workflow sends to a single `recipient_email`. For a proper subscriber list:
1. Store subscribers in a Sheets tab
2. After **Build Newsletter**, add a split-in-batches loop over subscribers
3. Call the SMTP send node once per subscriber

For large lists (100+), use a proper email provider (Mailchimp, ConvertKit, Resend) instead of direct SMTP — transactional SMTP has delivery limits and no unsubscribe management.

---

## Manual trigger

```bash
curl -X POST https://your-n8n.com/webhook/curate-newsletter \
  -H "Content-Type: application/json" \
  -d '{}'
```

Runs immediately using the current Config sheet settings.

---

## Raw article log

Every article fetched from Tavily is logged to the Raw Articles tab with the fetch timestamp and topic. Useful for:
- Auditing what sources are being found
- Identifying topics that consistently return poor content (adjust the search query)
- Building a longer-term archive of what was searched

---

## Changing the schedule

The newsletter fires every Monday at 6am. To change:
- Day: adjust `triggerAtDay` (0=Sunday, 1=Monday, etc.)
- Time: adjust `triggerAtHour` and `triggerAtMinute`
- Frequency: change `weeksInterval` to 2 for bi-weekly

---

## Limitations

- Tavily finds indexed web content from the past 7 days. Paywalled content, private newsletters, and unindexed content won't appear.
- Search quality depends on topic specificity. Broad topics like "technology news" return lower-quality results than specific ones like "Kubernetes release notes" or "Y Combinator batch analysis."
- The commentary is Claude's perspective based on the article title and snippet — it hasn't read the full article. For topics where nuance matters, you may want to review before sending.
- No deduplication across issues — the same article could appear in two consecutive weeks if it stays in Tavily's results. Add a "previously seen URLs" log to prevent this if it's a problem.

---

## Ideas

- [ ] Multi-recipient list: loop over a subscribers sheet instead of a single email
- [ ] RSS feed input: supplement Tavily with specific RSS feeds you want to monitor
- [ ] Topic rotation: swap topics week by week to cover a broader range without making each issue too scattered
- [ ] Click tracking: add UTM parameters to links and track which items get the most engagement

---

## License

MIT.
