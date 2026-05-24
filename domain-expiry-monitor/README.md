# domain-expiry-monitor

Domain expiry is one of those infrastructure problems that's completely preventable and still causes outages. A domain lapses because nobody updated the payment method, or the renewal email went to someone who left the company, or the registrar account had a different email than anyone checks. The site goes down, the business notices, and two frantic hours later someone figures out it was a $15/year renewal.

This checks every domain in your list daily, classifies expiry urgency (expired, critical 7 days, urgent 30 days, warning 60 days, notice 90 days), distinguishes between auto-renewing domains (lower risk) and manual ones (action required), and emails a prioritized alert only when domains need attention. Claude reads the full picture and writes a plain-language summary of what needs to happen today.

There's also a companion webhook to add new domains directly to the monitoring list without opening the sheet.

---

## What it does

**Daily scan (7:30am):**
- Loads all domains from Google Sheets
- Calculates days until expiry for each
- Classifies urgency: expired / critical / urgent / warning / notice
- Flags manual-renewal domains separately from auto-renew
- Claude summarizes what needs action today and why
- Emails formatted alert table to the tech team
- Silent if all domains are healthy (90+ days to expiry, or auto-renewing)

**Add domain (webhook `/add-domain`):**
- POST a domain name, expiry date, registrar, owner, auto-renew status
- Strips protocol and path — just saves the domain
- Adds to Google Sheets immediately
- Returns confirmation

---

## Stack

- **n8n** — daily scheduler + webhook
- **Google Sheets** — domain registry
- **Anthropic Claude** (claude-sonnet-4-20250514) — alert analysis
- **SMTP** — email alerts

---

## Setup

### 1. Create the Domains sheet

One tab: **Domains** — columns:
```
domain | expiry_date | auto_renew | registrar | owner | notes | last_alert_days | added_at
```

Fill in your domains. Key columns:
- `domain` — just the domain, e.g. `yourcompany.com`
- `expiry_date` — format `YYYY-MM-DD`
- `auto_renew` — TRUE or FALSE
- `registrar` — Namecheap, GoDaddy, Cloudflare, etc.
- `owner` — team or person responsible
- `last_alert_days` — updated manually when an alert is sent to prevent repeat pings

### 2. Environment variables

```
DOMAINS_SHEET_ID=your_google_sheet_id
FROM_EMAIL=infra@yourcompany.com
TECH_EMAIL=devops@yourcompany.com
```

### 3. Credentials

- **Google Sheets OAuth2**
- **Anthropic API** (LangChain node)
- **SMTP**

### 4. Import and activate

Import `workflow.json`, activate. Test by running the daily scanner manually.

---

## Urgency levels

| Status | Condition | Action |
|---|---|---|
| `EXPIRED` | Past expiry date | Urgent recovery needed |
| `CRITICAL` | ≤ 7 days | Renew immediately |
| `URGENT` | 8–30 days | Renew this week |
| `WARNING` | 31–60 days | Schedule renewal |
| `NOTICE` | 61–90 days | On your radar |
| `UNKNOWN` | No expiry date in sheet | Update the sheet |

Domains with `auto_renew = TRUE` still appear in alerts if they're critical or expired — auto-renew can fail (expired card, account issue), so critical domains always surface regardless.

---

## Adding a domain

```bash
curl -X POST https://your-n8n.com/webhook/add-domain \
  -H "Content-Type: application/json" \
  -d '{
    "domain": "newacquisition.com",
    "expiry_date": "2025-09-15",
    "auto_renew": false,
    "registrar": "Namecheap",
    "owner": "Infrastructure team",
    "notes": "Acquired from previous owner, update nameservers by June"
  }'
```

The webhook strips `https://`, `http://`, and any path — you can pass a full URL and it'll normalize to just the domain.

---

## Auto-renew vs manual

The most important column. Domains with `auto_renew = TRUE` are shown in the alert table but with lower urgency — the renewal should happen automatically if the registrar account is healthy. Domains with `auto_renew = FALSE` are specifically called out as needing manual action.

Claude's summary note always leads with the manual-renewal domains when they're at risk.

---

## When no alert is sent

The workflow only emails when at least one domain needs attention (any urgency level up to 90 days). If all your domains are healthy and auto-renewing with 90+ days to expiry, the daily check runs silently. No email until something approaches the threshold.

---

## Getting expiry dates

If you're starting fresh and don't know your expiry dates:
- Log into each registrar account and check the domain list
- Use `whois yourdomain.com` in a terminal — look for "Expiry Date" or "Registry Expiry Date"
- Many domain management tools (Cloudflare, etc.) show expiry dates in their dashboard

Update the sheet once and the monitor handles the rest.

---

## Multi-team setup

The `owner` column lets you track which team or person is responsible for each domain. The alert email shows owners in the table — useful for organizations where different teams manage different domains. For routing alerts to different teams automatically, add a lookup in the **Build Alert Email** node that groups domains by owner and sends separate emails.

---

## Limitations

- The workflow doesn't automatically look up expiry dates — you need to enter them in the sheet. To automate this, add a WHOIS lookup API call before the **Check Expiry Dates** node.
- The `last_alert_days` column is for preventing duplicate alerts but it's manually updated — there's no automatic write-back when an alert is sent. Add an update step after **Send Alert** if you want this automated.
- Alert is sent to a single email address. For Slack delivery, swap the email send node for a Slack node.

---

## Ideas

- [ ] WHOIS auto-lookup: before checking expiry dates, auto-fetch current expiry from WHOIS and update the sheet
- [ ] Slack delivery: post critical domain alerts to a #infrastructure channel
- [ ] SSL certificate monitoring: companion workflow that checks SSL cert expiry dates (typically 90-day Let's Encrypt certs or annual paid certs)
- [ ] Registrar login health check: periodic test that registrar accounts are accessible

---

## License

MIT.
