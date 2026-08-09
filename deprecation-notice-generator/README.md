# deprecation-notice-generator

A single deprecation email sent once tends to get missed. Effective deprecation communication uses a graduated sequence: an informative initial announcement with plenty of lead time, a more urgent reminder at the midpoint, and a clear final warning right before sunset. This generates all three notices in one call, calibrated in urgency, plus a migration checklist, FAQ, and internal talking points for support.

---

## What it does

Takes the deprecated item, item type, deprecation date, sunset date, replacement/alternative, reason, affected user estimate, and migration complexity. Claude produces:

- **Initial announcement** — subject, headline, full body: what's deprecated, why, timeline, migration path, where to get help
- **Reminder notice** — shorter, more urgent, for roughly the midpoint before sunset
- **Final warning** — urgent, clear about what breaks and when
- **Changelog entry** — concise changelog-style summary
- **Migration checklist** — specific steps for affected users
- **FAQ** — likely questions with answers
- **Internal talking points** — for support/CS team fielding customer questions

Can auto-send the initial announcement to an audience list immediately, or route the full set to a preview email for review first.

HTML output shows all three notices color-coded by urgency (blue → amber → red).

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-deprecation-notice \
  -H "Content-Type: application/json" \
  -d '{
    "product_name": "Flowdesk API",
    "deprecated_item": "Task API v1 (all endpoints under /api/v1/tasks)",
    "item_type": "api_endpoint",
    "deprecation_date": "2025-06-18",
    "sunset_date": "2025-09-18",
    "replacement_or_alternative": "Task API v2 (/api/v2/tasks) — same core functionality plus bulk operations and improved field types. See migration guide for the full diff.",
    "reason_for_deprecation": "v1 uses a data model that cannot support the new bulk operations and real-time collaboration features. v2 was designed to be extensible for these.",
    "affected_user_estimate": "Approximately 140 integrations currently using v1 endpoints, based on API traffic in the last 30 days",
    "migration_complexity": "moderate",
    "migration_guide_url": "https://docs.flowdesk.com/api/v1-to-v2-migration",
    "support_channel": "api-support@flowdesk.com or #api-help in our developer Slack",
    "audience_email_list": "api-developers@flowdesk.com",
    "reply_email": "platform@flowdesk.com"
  }'
```

**Required:** `deprecated_item`, `deprecation_date`, `replacement_or_alternative`

---

## Item types

`api_endpoint`, `feature`, `integration`, `library_version`, `entire_product`, `sdk_method`, `config_option`

---

## Three-notice sequence

Send the `initial_announcement` at deprecation time (maximum lead time). Send the `reminder_notice` at roughly the midpoint between announcement and sunset. Send the `final_warning` in the last 1–2 weeks before sunset. Each is written with escalating urgency but consistent facts — no surprises, just increasing clarity that the deadline is approaching.

---

## Internal talking points

The `internal_talking_points` field gives support and CS teams consistent language to use when customers ask about the deprecation — avoiding the situation where different reps give different explanations for why something is being removed.

---

## Limitations

- The initial notice can be auto-sent via `audience_email_list`, but the reminder and final warning are generated together and need to be scheduled/sent separately at the appropriate later dates — this agent doesn't handle scheduling.
- Migration complexity assessment (`migration_complexity`) is used to calibrate tone and checklist detail; pass an honest assessment for the best result.

---

## License

MIT.
