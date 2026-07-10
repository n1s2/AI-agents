# incident-communication-writer

Incident communication done poorly erodes trust faster than the incident itself. Vague updates ("we're aware of issues"), over-promising ETAs, or radio silence all tell customers the team isn't in control. This writes three simultaneous communication artifacts — customer email, status page update, and internal support brief — calibrated to the incident status, severity, and what's actually known.

---

## What it does

Takes incident details: title, status (investigating/identified/monitoring/resolved), severity, impact description, affected services, current knowledge, resolution if known, and ETA. Claude writes all requested communications simultaneously:

- **Customer email** — subject + body: honest, clear, not overly apologetic. What happened, who's affected, what's being done, ETA or next update time, workarounds if any.
- **Status page update** — headline (under 80 chars) + 1–3 sentence factual update
- **Internal update** — for support/CS team: includes technical context, what to tell customers who contact support, what NOT to say publicly yet
- **Social post** — under 280 chars for Twitter/X if needed
- **What not to say** — phrases and commitments to avoid
- **Next update guidance** — when and what to communicate next

Can optionally send each communication to its actual recipient list immediately.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/write-incident-comms \
  -H "Content-Type: application/json" \
  -d '{
    "incident_id": "INC-2025-047",
    "incident_title": "Payment processing degraded",
    "incident_status": "identified",
    "severity": "p1",
    "product_name": "Flowdesk",
    "started_at": "2025-05-29 14:32 UTC",
    "affected_services": ["Checkout", "Billing API", "Subscription management"],
    "impact_description": "Some customers are unable to complete purchases or upgrade their plan. Existing subscriptions are not affected — no service interruption for current paid users.",
    "current_knowledge": "Root cause identified: a database connection pool exhaustion in the payment service caused by a deploy at 14:15 UTC. We are rolling back the deployment and expect full restoration shortly.",
    "eta": "30-45 minutes from now",
    "audiences": ["customers", "status_page", "internal"],
    "tone_guidance": "Transparent and calm. Acknowledge the problem directly without being alarmist. Customers value honesty.",
    "customer_email_list": "incidents@flowdesk.com",
    "internal_email_list": "support-team@flowdesk.com",
    "reply_email": "ops@flowdesk.com"
  }'
```

**Required:** `incident_title`, `incident_status`, `impact_description`

---

## Status calibration

Each status produces different communication:

- **Investigating** — honest about not knowing the cause yet, commits to next update time
- **Identified** — explains root cause in plain language, gives realistic ETA
- **Monitoring** — confirms fix deployed, explains what's being monitored before declaring resolved
- **Resolved** — what happened, what was fixed, what prevented it from being caught sooner

The same incident should produce different communications at each status change.

---

## What not to say

Claude always includes a `what_not_to_say` list specific to this incident. Common items: don't promise ETAs you can't keep, don't say "minor issue" when there's real user impact, don't use "we apologize for any inconvenience" as a substitute for substance.

---

## Sending options

Pass recipient lists to auto-send:
- `customer_email_list` — sends customer email immediately (use with caution; review first)
- `internal_email_list` — sends internal brief to support team immediately
- `reply_email` — sends full HTML doc with all communications for review

For a review-before-sending workflow, pass only `reply_email` and review the HTML doc before triggering actual sends.

---

## License

MIT.
