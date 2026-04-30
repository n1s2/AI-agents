# outage-postmortem-generator

Writing postmortems after an incident is important and also the last thing anyone wants to do. The oncall engineer just spent 4 hours fighting a production fire, it's 2am, and now they have to write a coherent document for leadership while the adrenaline is still wearing off.

The result is usually either (a) a wall of Slack messages copy-pasted into a doc, or (b) a vague 3-paragraph thing that says "we'll add monitoring" at the end and then nobody does.

This takes the raw incident data — title, severity, timeline, summary, affected systems — sends it to Claude with a prompt that enforces a proper blameless postmortem structure, and returns a formatted doc. It optionally emails it and/or posts it to Slack.

Claude doesn't make up root causes. It works with what you give it. The better your timeline and summary, the better the output. Garbage in, garbage out — but even a rough summary tends to produce something more structured and useful than what a tired engineer writes from scratch.

---

## What it does

1. Accepts a POST with incident metadata + timeline
2. Calculates duration automatically from timestamps
3. Sends to Claude with a staff-engineer-level prompt enforcing blameless postmortem structure
4. Returns clean markdown with: summary, impact, timeline, root cause, contributing factors, what went well, what could've been better, action items table, lessons learned
5. Optionally emails the postmortem as formatted HTML
6. Optionally posts to a Slack channel
7. Returns the raw markdown in the webhook response

---

## Stack

- **n8n** — webhook + workflow
- **Anthropic Claude** (claude-opus-4-5) — document generation
- **SMTP** — email delivery (optional)
- **Slack** — channel notification (optional)

---

## Setup

### 1. Environment variables

```
FROM_EMAIL=postmortem@yourcompany.com
```

### 2. Credentials

- **Anthropic API** — LangChain node
- **SMTP** — only needed if using `notify_email`
- **Slack API** — only needed if using `notify_slack_channel`

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/postmortem \
  -H "Content-Type: application/json" \
  -d '{
    "incident_title": "Checkout API Timeout Cascade",
    "severity": "SEV1",
    "started_at": "2025-04-23T02:14:00Z",
    "resolved_at": "2025-04-23T04:47:00Z",
    "team": "Payments",
    "incident_commander": "On-call SRE",
    "affected_systems": ["checkout-api", "payment-processor", "order-service"],
    "impact_description": "100% of checkout attempts failed for 2h33m. Estimated 4,200 failed transactions. Approx $140k revenue impact.",
    "summary": "A deploy of checkout-api v2.4.1 introduced a connection pool misconfiguration. Under normal load the pool held steady, but a routine DB maintenance window reduced available connections by 40%, causing the pool to exhaust. Timeouts cascaded to payment-processor which had no circuit breaker. Order-service began queuing requests until memory pressure caused OOM restarts. Rollback to v2.4.0 resolved within 8 minutes of identification.",
    "timeline": [
      { "time": "02:14", "event": "Checkout error rate alert fires (threshold: 5%, actual: 94%)" },
      { "time": "02:17", "event": "Oncall acknowledges alert, begins investigation" },
      { "time": "02:31", "event": "Connection pool exhaustion identified in checkout-api logs" },
      { "time": "02:38", "event": "Recent deploys reviewed — v2.4.1 shipped 3h prior" },
      { "time": "02:45", "event": "DB maintenance window correlated as contributing trigger" },
      { "time": "02:52", "event": "Decision made to rollback to v2.4.0" },
      { "time": "03:00", "event": "Rollback complete, checkout error rate drops to 0.2%" },
      { "time": "04:47", "event": "All systems confirmed stable, incident resolved" }
    ],
    "notify_email": "engineering@yourcompany.com",
    "notify_slack_channel": "#incidents"
  }'
```

---

## Supported severity values

`SEV1`, `SEV2`, `SEV3`, `P0`, `P1`, `P2`, `critical`, `high`, `medium`

These affect the color coding in the HTML email header. Map them to whatever your org uses.

---

## Timeline format

Each timeline entry can be either:

```json
{ "time": "02:14", "event": "What happened" }
```

Or just a string:

```json
"02:14 — Checkout error rate alert fires"
```

Both work. 30 entries max — if your timeline is longer than that, summarize the middle section.

---

## Output format

The response always contains `markdown` — the raw postmortem text. If you want to save it somewhere (Confluence, Notion, GitHub), you can pipe the response to your tool of choice.

```bash
# save to file
curl ... | jq -r '.markdown' > postmortem-2025-04-23.md

# push to GitHub
gh gist create --filename postmortem-checkout-timeout.md - <<< "$(curl ... | jq -r '.markdown')"
```

---

## Integrating with PagerDuty / incident.io

If you use PagerDuty or incident.io, you can trigger this webhook automatically when an incident is resolved. Both support webhook-out on status change. Pass the incident data as the body and you get a postmortem draft waiting in Slack before the retrospective meeting.

For PagerDuty: use their webhooks v3 → hit a small Lambda/function that reformats the PD payload → calls this webhook.

---

## Quality of the output

Claude writes what you give it. The minimum viable input is `summary` (a paragraph) + `timeline` (5+ entries). With that you'll get a reasonable draft.

The output improves significantly with:
- A clear `impact_description` with numbers
- A timeline that includes the detection event, not just the fix
- An honest `summary` that mentions what specifically failed, not just "there was an issue"

The action items table always has suggested owners by role (e.g. "Oncall SRE", "Platform team") since Claude doesn't know your org chart. Fill in real names before publishing.

---

## Known limitations

- Claude will occasionally generate action items that are too vague ("improve monitoring") — worth reviewing before publishing
- If your summary is sparse, the root cause section will be thin. That's a feature, not a bug — it shouldn't invent causes.
- The Slack output truncates at 2800 characters. Long postmortems get cut off in Slack but arrive complete via email and the webhook response.

---

## Blameless postmortem philosophy

The system prompt explicitly enforces blameless writing. Claude won't frame things as "engineer X failed to" — it focuses on system and process failures. If you want to add blame somewhere, that's your own business and you can edit the doc after.

---

## Ideas

- [ ] Confluence API integration to auto-publish to a postmortems space
- [ ] Automatic Jira ticket creation for each action item
- [ ] Template selector (customer-facing vs internal vs exec summary)
- [ ] Historical postmortem analysis ("what themes recur across incidents")

---

## License

MIT.
