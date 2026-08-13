# on-call-handoff-generator

A Slack message that just says "quiet shift, nothing major" loses the small anomaly the outgoing engineer noticed but didn't think was worth a full incident — the kind of thing that becomes an incident three hours into the next shift. This structures on-call handoffs so nothing falls through the cracks: incidents with current state and required follow-up, watch items with specific trigger conditions, and questions the incoming engineer should confirm before the outgoing engineer signs off.

---

## What it does

Takes outgoing/incoming engineer names, shift period, incidents during the shift, watch items, planned maintenance, deployments, and system health notes. Claude produces:

- **Handoff summary** and **overall status** — quiet/normal/elevated_attention/active_concern
- **Incidents recap** — each with what happened, current state (resolved/monitoring/ongoing), and specific action needed from the incoming engineer if any
- **Active watch items** — each with why it's being watched, what would trigger action, and priority
- **System state summary** — honest assessment going into the next shift
- **Upcoming events** — scheduled deployments, maintenance, or known risk windows
- **Questions for incoming** — specific things to confirm or investigate
- **Quick reference** — anything currently paging, recent deploys to be aware of

HTML handoff document with status badge, incident cards, and watch items with trigger conditions clearly stated.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-oncall-handoff \
  -H "Content-Type: application/json" \
  -d '{
    "outgoing_engineer": "Sara Kim",
    "incoming_engineer": "Tom Walsh",
    "shift_period": "June 18, 9pm - June 19, 9am",
    "reply_email": "sara@flowdesk.com",
    "incidents_this_shift": [
      {"id": "INC-088", "title": "Elevated 500s on /api/v2/tasks", "severity": "medium", "status": "resolved", "notes": "Caused by a connection pool exhaustion during a traffic spike. Rolled back a config change from earlier deploy. Resolved in 22 minutes."}
    ],
    "watch_items": [
      "Database CPU has been hovering at 65-70% since 3am, higher than typical overnight baseline of 40-50%. Not alerting yet but worth watching if it climbs further.",
      "One customer (Beacon Logistics) reported slow page loads around 2am via support chat, but our metrics did not show anything unusual at that time. Possibly client-side or ISP issue, but flagging in case it recurs."
    ],
    "planned_maintenance": "Scheduled DB index rebuild tomorrow 2pm-3pm, should be transparent to users but will show elevated query latency during the window",
    "deployments_this_shift": "One hotfix deploy at 11:45pm to address the connection pool issue (rollback of earlier config change)",
    "system_health_notes": "All services green except the DB CPU watch item noted above. No customer-reported incidents beyond the one noted.",
    "open_questions": "Should we increase the connection pool size permanently or was this a one-time traffic spike? Need to check with growth team about whether that marketing campaign traffic will repeat."
  }'
```

**Required:** `outgoing_engineer`, `incoming_engineer`, `shift_period`

---

## Watch items with trigger conditions

The most valuable part of a handoff isn't "keep an eye on the database" — it's a specific trigger: "if DB CPU exceeds 80%, page the DBA on-call." Claude's `what_would_trigger_action` field forces this specificity, so the incoming engineer knows exactly when to escalate rather than having to make a judgment call with less context than the outgoing engineer had.

---

## Overall status calibration

`quiet`, `normal`, `elevated_attention`, `active_concern` — this gives leadership and the incoming engineer an immediate sense of how seriously to take the shift, before reading the details.

---

## Limitations

- The handoff is only as good as the notes provided. Encourage engineers to note anomalies even if they don't rise to "incident" status — the watch items section exists specifically to capture the borderline stuff.
- This generates a handoff document — it doesn't integrate with your actual PagerDuty/Opsgenie rotation. Trigger it via a scheduled workflow at shift-change time, populated from your incident tracker.

---

## License

MIT.
