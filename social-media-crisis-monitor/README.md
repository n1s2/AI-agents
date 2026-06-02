# social-media-crisis-monitor

Brand crises move fast. By the time someone sees a tweet going viral or a Reddit thread gaining traction, the story has already been picked up by news outlets. The window to get ahead of it is measured in hours, not days.

This scans the web every 30 minutes for negative mentions, complaints, scandals, and controversies involving your brand. It scores the severity, classifies it as clear / watch / elevated / critical, and alerts the PR team when something needs attention — with Claude's read on whether it's genuine noise or an actual threat and what to do about it right now.

You can monitor multiple brands from a single Google Sheet config.

---

## What it does

**Automated scan (every 30 minutes):**
- Loads all active brands from Google Sheets config
- For each brand: runs two Tavily searches — one for negative signals (complaint, scandal, controversy, etc.) and one for general news
- Scores each result for negative and positive keywords
- Classifies crisis level: `clear`, `watch`, `elevated`, `critical`
- If any signals exist: Claude assesses whether it's noise or a genuine threat and gives specific PR response recommendations
- Logs to Google Sheets Incidents tab
- Sends email and/or Slack alert to the brand's configured contacts

**Manual check (webhook `/crisis-check`):**
- POST to trigger an immediate scan of all brands (or a specific brand)
- Returns the current crisis level

---

## Stack

- **n8n** — 30-minute scheduler + webhook
- **Tavily API** — web monitoring (two searches per brand per cycle)
- **Google Sheets** — brand config + incident log
- **Anthropic Claude** (claude-sonnet-4-20250514) — crisis assessment
- **SMTP** — alert emails
- **Slack** — alert messages

---

## Setup

### 1. Create the Google Sheet

Two tabs:

**Tab: Brands** — columns:
```
brand_name | search_terms | alert_email | alert_slack | crisis_threshold | industry | competitors | enabled
```

**Tab: Incidents** — columns:
```
checked_at | brand | crisis_level | neg_score | total_mentions | top_item
```

**Brands tab explained:**
- `brand_name` — your company/brand name
- `search_terms` — comma-separated list of terms to search (e.g. `YourBrand, @YourBrand, YourProduct`) — defaults to brand_name if blank
- `crisis_threshold` — negative score at which to trigger elevated alert (default 5)
- `enabled` — set to FALSE to pause monitoring for a brand

### 2. Environment variables

```
CRISIS_SHEET_ID=your_google_sheet_id
TAVILY_API_KEY=tvly-your-key
FROM_EMAIL=monitor@yourcompany.com
```

### 3. Credentials

- **Google Sheets OAuth2**
- **Anthropic API** (LangChain node)
- **SMTP**
- **Slack API**

### 4. Import and activate

Import `workflow.json`, activate. Test by triggering the scheduler manually.

---

## Crisis levels

| Level | Condition | Action |
|---|---|---|
| `CLEAR` | No negative signals | No alert sent |
| `WATCH` | Some negative mentions, low score | Alert sent, monitor |
| `ELEVATED` | Score ≥ threshold | Alert sent, review needed |
| `CRITICAL` | Score ≥ 2× threshold | Alert sent, immediate action |

---

## Signal scoring

The scoring system checks for negative keywords in article titles and content:

**Negative keywords:** scandal, controversy, lawsuit, boycott, outrage, backlash, complaint, problem, fail, crisis, recall, breach, hack, fraud, mislead, discriminat, harass, abuse, fire, resign, bankrupt, shutdown

Each keyword match in a relevant article adds 1 to that article's negative score. Total negative score across all articles determines crisis level.

**Positive keywords** (offsets score): award, launch, partnership, growth, funding, expansion, recognition, milestone

---

## Crisis threshold

`crisis_threshold` in the Brands sheet sets when elevated alerts fire. Default is 5 — meaning at least 5 negative keyword hits across all articles before alerting.

For brands in sensitive industries (pharma, finance, food safety) or during a known risk period, lower this to 2–3 for earlier warning. For brands with high ambient noise (large consumer companies that always have some complaints), raise it to 8–10.

---

## Search terms

The `search_terms` column takes a comma-separated list. Use this to catch:
- Brand name variants (YourBrand, Your Brand)
- Product names
- Social handles (@yourbrand)
- CEO name (if executive reputation monitoring is needed)
- Common misspellings

Example: `Acme Corp, AcmeCorp, @acmecorp, Acme Widget`

---

## Multiple brands

Add one row per brand to the Brands sheet. Each brand can have its own:
- Alert email (routes to the right PR team)
- Slack channel (#acme-alerts vs #brandx-alerts)
- Crisis threshold
- Search terms

All brands are checked every 30 minutes in sequence.

---

## What Claude assesses

After scoring, Claude gets the full picture and answers three things:
1. What's actually happening — is this a new complaint, a trending story, a recycled old issue?
2. Is it noise or a genuine threat — isolated complaints are different from coordinated criticism
3. Specific next steps for the comms team — whether to monitor, draft a holding statement, or activate full crisis response

---

## Manual trigger

```bash
# Check all brands immediately
curl -X POST https://your-n8n.com/webhook/crisis-check \
  -H "Content-Type: application/json" \
  -d '{}'

# Check a specific brand
curl -X POST https://your-n8n.com/webhook/crisis-check \
  -H "Content-Type: application/json" \
  -d '{ "brand_name": "Acme Corp" }'
```

---

## Limitations

- Tavily searches indexed web content — it won't catch private Twitter DMs, closed Facebook groups, or forums that block indexing. It's strongest for news, Reddit, and public social content that gets indexed quickly.
- The keyword scoring is simple and can produce false positives (a brand being mentioned in a story about industry-wide fraud, for example). Claude's analysis is the second filter — it distinguishes between actual brand involvement and tangential mentions.
- Two searches per brand per 30-minute cycle costs API credits. For brands with high ambient news volume, you may want to reduce to hourly checks.
- The monitor looks back 24 hours by default (Tavily `days: 1`). It may re-surface the same story across multiple cycles until it ages out.

---

## Ideas

- [ ] Sentiment trend chart: track daily average scores in the Incidents tab to visualize sentiment trends over time
- [ ] Competitor monitoring: add competitor brand names to the search config and track their crises for context
- [ ] Draft holding statement: when CRITICAL is triggered, auto-generate a draft holding statement for PR review
- [ ] Escalation tiers: page an on-call person via PagerDuty or SMS when CRITICAL fires outside business hours

---

## License

MIT.
