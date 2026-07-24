# changelog-digest-sender

One set of release notes doesn't work for all audiences. Customers care about what they can do now that they couldn't before. Support teams need to know what to watch for and what to tell customers. Developers need API changes and migration steps. Executives want one sentence and the business impact. This takes a list of raw changes and generates audience-specific versions, then optionally sends each to the right list.

---

## What it does

Takes product name, version, release date, and an array of changes (with type, impact, description, and affected areas). Generates native versions for each requested audience:

- **Customers** — benefit-led headline, intro, highlights by change type (new/improved/fixed), closing with next step. Email subject + preview text included.
- **Internal (support/CS)** — summary of what shipped, per-change implementation notes, what to watch for, rollout notes
- **Developers** — API changes with migration required flag and migration steps, deprecations
- **Executives** — one-liner, 2–3 key business wins, metrics to watch

Can auto-send each version to its recipient list simultaneously, or route everything to a preview email for review first.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/send-changelog-digest \
  -H "Content-Type: application/json" \
  -d '{
    "product_name": "Flowdesk",
    "release_version": "2.8.0",
    "release_date": "2025-06-03",
    "period_covered": "May 2025",
    "brand_voice": "Friendly and direct — we celebrate wins without overselling",
    "target_audiences": ["customers", "internal", "developers"],
    "customer_email_list": "updates@flowdesk.com",
    "internal_email_list": "support@flowdesk.com",
    "reply_email": "product@flowdesk.com",
    "changes": [
      {"id": "FD-1201", "title": "Notion bidirectional sync", "type": "new", "impact": "high", "affected_areas": ["integrations", "tasks"], "description": "Users can now connect their Notion workspace and sync tasks bidirectionally. Changes in Flowdesk appear in Notion within 30 seconds and vice versa. Supports pages and databases.", "ticket_ref": "FD-1201"},
      {"id": "FD-1189", "title": "Safari iOS logout bug fix", "type": "fixed", "impact": "medium", "affected_areas": ["auth", "mobile"], "description": "Fixed a bug where Safari on iOS would log users out unexpectedly when switching tabs. Root cause was a session cookie SameSite misconfiguration.", "ticket_ref": "FD-1189"},
      {"id": "FD-1195", "title": "Bulk task assignment", "type": "new", "impact": "high", "affected_areas": ["tasks", "workflow"], "description": "Select multiple tasks and assign them all at once. Supports up to 100 tasks per bulk action. Available from the task list view via checkbox selection.", "ticket_ref": "FD-1195"},
      {"id": "FD-1178", "title": "Task API v2 — assignee field change", "type": "breaking", "impact": "high", "affected_areas": ["api"], "description": "The assignee field in Task API v2 now returns a user object instead of a string ID. Existing integrations using /api/v2/tasks will need to update their assignee handling.", "ticket_ref": "FD-1178"}
    ]
  }'
```

**Required:** `product_name`, `changes`

---

## Audiences

`customers`, `internal`, `developers`, `executives` — request any subset. Each gets a completely different version of the same release notes.

---

## Sending options

- `customer_email_list` — sends customer version immediately
- `internal_email_list` — sends internal version immediately
- `reply_email` — sends full preview HTML with all versions for review

For a review-before-sending workflow, pass only `reply_email` and check all versions before triggering the actual sends.

---

## Breaking changes

Changes with `type: "breaking"` are flagged prominently in the developer version with migration steps. They're excluded from the customer-facing version (customers shouldn't see internal API changes) and mentioned carefully in the internal version so support knows what's changing.

---

## Limitations

- Customer email is sent to a single list address — for per-user personalization, pipe the customer version through an email platform (Mailchimp, Customer.io) rather than using the SMTP send directly.
- The agent generates one release digest per call. For weekly digests that aggregate multiple releases, pass all changes from the period in one call.

---

## License

MIT.
