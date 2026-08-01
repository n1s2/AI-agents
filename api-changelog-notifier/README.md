# api-changelog-notifier

API changelogs that just say "updated authentication" or "fixed an issue with the /users endpoint" don't help developers plan their migration work. A good API changelog is specific about what changed, explicit about the impact on existing code, and gives exact migration steps with before/after code examples. This generates developer-grade changelogs and sends them to your developer mailing list and Slack channel simultaneously.

---

## What it does

Takes API name, version, release date, and an array of changes (each with type, endpoint, field, description, migration steps, and deprecation dates). Claude generates:

- **Email subject line** with BREAKING/SECURITY flags where appropriate
- **Plain-text summary** — what changed in plain English, what developers need to do
- **Sections** grouped by type (breaking_changes/deprecations/new_features/improvements/bug_fixes/security) — each change with: technical description, impact on existing code, migration steps, and before/after code examples
- **Slack message** — concise under-200-char Slack summary
- **Action required** flag and deadline
- **Affected developer count estimate**

Can simultaneously: send HTML email to developer mailing list, post to Slack via webhook, and send full preview to reply email.

HTML rendered with dark developer aesthetic (dark header, monospace code blocks).

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP, Slack webhooks (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/notify-api-changelog \
  -H "Content-Type: application/json" \
  -d '{
    "api_name": "Flowdesk API",
    "version": "2.0",
    "release_date": "2025-07-01",
    "migration_guide_url": "https://docs.flowdesk.com/api/v2-migration",
    "docs_url": "https://docs.flowdesk.com/api",
    "support_email": "api@flowdesk.com",
    "developer_email_list": "api-users@flowdesk.com",
    "slack_webhook": "https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK",
    "reply_email": "platform@flowdesk.com",
    "changes": [
      {
        "id": "BC-001",
        "type": "breaking",
        "severity": "critical",
        "endpoint": "GET /api/v2/tasks",
        "field": "assignee",
        "description": "The assignee field now returns a full user object instead of a string user ID",
        "migration_steps": "Update any code that reads task.assignee as a string to use task.assignee.id for the ID and task.assignee.name for the display name",
        "affected_versions": "v1.x",
        "example_before": "{ assignee: \"usr_abc123\" }",
        "replaced_by": "{ assignee: { id: \"usr_abc123\", name: \"Tom Walsh\", email: \"tom@...\" } }"
      },
      {
        "id": "DEP-001",
        "type": "deprecated",
        "severity": "high",
        "endpoint": "GET /api/v1/tasks",
        "description": "The v1 Tasks endpoint is deprecated. All v1 endpoints will be removed on 2025-10-01.",
        "migration_steps": "Migrate to /api/v2/tasks. See migration guide for full diff.",
        "deprecation_date": "2025-10-01",
        "replaced_by": "GET /api/v2/tasks"
      },
      {
        "id": "NEW-001",
        "type": "new_endpoint",
        "severity": "low",
        "endpoint": "POST /api/v2/tasks/bulk-assign",
        "description": "New endpoint for assigning up to 100 tasks in a single request. Accepts an array of task IDs and a user ID.",
        "migration_steps": "No migration required — new capability"
      }
    ]
  }'
```

**Required:** `api_name`, `changes`

---

## Change types

`breaking`, `deprecated`, `new_endpoint`, `new_field`, `behavior_change`, `performance`, `security`, `bug_fix`, `rate_limit`, `auth`

Breaking changes and security changes are automatically flagged in the email subject and rendered at the top of the notification.

---

## Before/after code examples

Pass `example_before` and `replaced_by` (or `example_after`) per change and Claude includes them as monospace before/after blocks in the notification. These are the most useful part for developers — seeing the exact change in code rather than having to infer it from a description.

---

## Multi-channel delivery

- `developer_email_list` — sends HTML email to your developer mailing list immediately
- `slack_webhook` — posts the Slack summary to your developer channel
- `reply_email` — sends full preview for review before broadcasting

Pass any combination. All three can fire simultaneously.

---

## Limitations

- Code examples are passed as strings and rendered as-is. Format them as you want them to appear.
- For very large APIs with many simultaneous changes, consider grouping changes into multiple calls by type (all breaking changes in one notification, all new features in another) to avoid overwhelming developers with a single giant email.

---

## License

MIT.
