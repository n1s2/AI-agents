# subscription-churn-detector

By the time a customer cancels, you've usually already lost them. The signals were there weeks earlier — login gaps, declining usage, a failed payment, a support ticket that didn't get resolved well. The problem is nobody was looking.

This runs every morning, scores every active subscriber against a set of churn risk signals, and emails the CS team a prioritized report of who's at risk, how much MRR is on the line, and what Claude thinks the highest-priority interventions are based on the actual signal patterns.

The scoring is the useful part. A cancel attempt is worth 40 points. A plan downgrade is 25. No login in 30 days is 25. A payment failure is 15–30 depending on how many. Scores are additive — a customer with declining usage, an NPS of 5, and a renewal in 10 days gets flagged high risk even if no single signal looks alarming on its own.

---

## What it does

**Daily scan (every day 7am):**
- Loads all active subscribers from Google Sheets
- Scores each subscriber across 8 risk signals: login recency, usage score, payment failures, support ticket volume, NPS score, plan downgrade, cancel attempt, renewal proximity
- Filters to anyone above a minimum threshold, sorted by risk score
- Flags as high / medium / low risk
- Calculates total MRR at risk
- Claude analyzes the at-risk cohort: top 2–3 interventions today, pattern observations, systemic fix if signals suggest a product or process issue
- Emails formatted report to CS team — only when there are at-risk accounts
- Returns full JSON via webhook response too

**Manual check (webhook):**
- POST to trigger a fresh scan any time, optionally scoped to a single customer_id

---

## Stack

- **n8n** — daily scheduler + webhook
- **Google Sheets** — subscriber data
- **Anthropic Claude** (claude-sonnet-4-20250514) — churn analysis
- **SMTP** — CS team email

---

## Setup

### 1. Create the Subscribers sheet

One tab: **Subscribers** — columns:

```
customer_id | customer_name | customer_email | plan | mrr | currency | status | last_login_date | weekly_usage_score | support_tickets_30d | payment_failures_90d | nps_score | plan_downgraded | cancel_attempt | next_renewal_date | last_payment_date | csm | notes
```

**Key columns explained:**
- `status` — only `active` rows are scanned; set to `churned` or `cancelled` when they leave
- `weekly_usage_score` — 0–100, however you define usage for your product (logins × actions, feature adoption score, etc.)
- `payment_failures_90d` — count of failed payment attempts in last 90 days
- `plan_downgraded` — TRUE/FALSE
- `cancel_attempt` — TRUE/FALSE — did they click cancel or contact support to cancel
- `nps_score` — 0–10, blank if not yet surveyed

### 2. Environment variables

```
CHURN_SHEET_ID=your_google_sheet_id
FROM_EMAIL=cs@yourcompany.com
CS_TEAM_EMAIL=cs-team@yourcompany.com
```

### 3. Credentials

- **Google Sheets OAuth2**
- **Anthropic API** (LangChain node)
- **SMTP**

### 4. Import and activate

Import `workflow.json`, activate. Test by running the daily scan node manually.

---

## Risk signal scoring

| Signal | Points |
|---|---|
| No login in 30+ days | 25 |
| No login in 14–30 days | 10 |
| Usage score < 20 | 20 |
| Usage score 20–40 | 10 |
| 2+ payment failures | 30 |
| 1 payment failure | 15 |
| 3+ support tickets in 30 days | 15 |
| NPS 0–6 (detractor) | 20 |
| NPS 7 (passive) | 8 |
| Plan downgraded | 25 |
| Cancel attempt logged | 40 |
| Renewal in ≤14 days + score >20 | 10 |

**Risk levels:**
- **High:** score ≥ 60
- **Medium:** score 35–59
- **Low:** score 20–34
- Below 20: not flagged

Adjust thresholds and point values in the **Score Churn Risk** node to match your product's behavior patterns.

---

## Keeping the sheet current

The workflow is only as good as the data. Fields that need regular updates:
- `last_login_date` — ideally synced from your product database daily
- `weekly_usage_score` — computed from your analytics and written back
- `payment_failures_90d` — synced from your payment provider (Stripe, etc.)
- `nps_score` — updated after each NPS survey response

If you use Stripe, you can build a companion n8n workflow that listens for `payment_intent.payment_failed` webhooks and increments the failures column automatically.

---

## What Claude looks at

Claude gets the full at-risk list with signal breakdowns and writes three things:
1. The 2–3 highest-priority interventions today — specific customers, specific angles based on their signals
2. A pattern observation — e.g. "five of the eight high-risk accounts have payment failures, suggesting a billing flow issue rather than product dissatisfaction"
3. A systemic fix if the signals point to a product or process problem rather than individual account issues

This is the part that saves time. Instead of reading 20 rows and figuring out where to start, the CS team gets a direct answer.

---

## Triggering manually

```bash
curl -X POST https://your-n8n.com/webhook/churn-check \
  -H "Content-Type: application/json" \
  -d '{}'
```

Or for a specific customer:

```bash
curl -X POST https://your-n8n.com/webhook/churn-check \
  -H "Content-Type: application/json" \
  -d '{ "customer_id": "cust_8472" }'
```

---

## Limitations

- The scoring is heuristic-based. It works well for typical SaaS patterns but you should tune the weights for your specific product — a tool used weekly behaves differently from one used daily.
- If your subscriber data isn't being updated regularly, the signals will be stale and the scores meaningless. The workflow is a good prompt to build proper data pipelines if you don't have them.
- The report shows up to 20 at-risk accounts in the email table. If you have more, they're still in the JSON response.

---

## Ideas

- [ ] Automated outreach trigger: for high-risk accounts, auto-draft a personalized save email using the signal data
- [ ] Stripe webhook integration: auto-update payment failures in real time
- [ ] Cohort comparison: compare this week's at-risk list to last week's — who got saved, who churned, who got worse
- [ ] CSM assignment routing: auto-assign high-risk accounts to a CSM based on account size or region

---

## License

MIT.
