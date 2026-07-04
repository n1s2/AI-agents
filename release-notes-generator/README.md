# release-notes-generator

Writing release notes is the last step of a sprint and usually the most rushed. Engineers write them in engineering language, product managers write them too vaguely, and nobody writes three versions: one for users, one for developers, one for CHANGELOG.md. This takes a structured list of changes and produces all three, plus an email subject line and tweet.

---

## What it does

Takes version number, release date, product name, and a list of changes (each with type, title, description, audience, and optionally a Jira ticket). Claude generates:

- Release headline (1-sentence summary of the most important change)
- **User-facing notes**: friendly header, 2–3 sentence non-technical summary, grouped sections (new features/improvements/bug fixes/important notes) with benefit-led titles and plain-language descriptions
- **Technical notes**: developer-oriented summary, breaking changes with migration notes, technical change details with ticket references
- **CHANGELOG.md entry**: complete markdown ready to paste
- Email subject line for the release announcement
- Tweet under 280 characters

Internal-only changes (flagged with `internal: true`) are excluded from user-facing and technical output but can be included in a separate section if `include_internal_notes` is true.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-release-notes \
  -H "Content-Type: application/json" \
  -d '{
    "version": "v2.5.0",
    "release_date": "2025-05-25",
    "product_name": "Flowdesk",
    "tone": "clear_and_friendly",
    "output_format": "all",
    "reply_email": "dev@flowdesk.com",
    "changes": [
      {"type": "feature", "title": "Notion bidirectional sync", "description": "Tasks created in Flowdesk now sync to a connected Notion database and vice versa. Requires OAuth connection in Settings > Integrations.", "audience": "all", "jira_ticket": "FD-1201"},
      {"type": "improvement", "title": "Bulk task assignment", "description": "Select multiple tasks and assign them to a team member in one action. Accessible via the checkbox selector on the task list.", "audience": "end_users"},
      {"type": "bugfix", "title": "Fixed logout bug on Safari iOS", "description": "Some Safari iOS users were being logged out after 15 minutes due to an incorrect session token expiry. This is now resolved.", "audience": "end_users", "jira_ticket": "FD-1189"},
      {"type": "breaking_change", "title": "Webhook payload format updated", "description": "task.updated webhook events now include a previous_values field containing the field values before the update. Integrations relying on the old payload shape will need to be updated.", "audience": "developers", "jira_ticket": "FD-1178"},
      {"type": "security", "title": "Updated session token rotation", "description": "Session tokens now rotate on each request rather than at fixed intervals.", "internal": true}
    ]
  }'
```

**Required:** `version`, `changes` (non-empty array with at least one titled change)

---

## Change types

`feature`, `improvement`, `bugfix`, `breaking_change`, `deprecation`, `performance`, `security`

Breaking changes get called out prominently in both user-facing and technical notes with migration guidance.

---

## Audience targeting

Each change can be tagged with `audience: "all" | "developers" | "admins" | "end_users"`. Claude uses this to determine where changes appear in each output format — a developer-only breaking change won't lead the user-facing notes.

---

## Internal changes

Mark changes with `internal: true` to exclude them from public-facing output. Pass `include_internal_notes: true` to see them in a separate section in the generated doc (useful for internal communication but not customer-facing).

---

## Output formats

`all` — generates user-facing, technical, and CHANGELOG.md
`user_facing` — only the user-facing section
`technical` — only the technical/developer section
`changelog_md` — only the markdown

---

## Limitations

- Quality depends on the input descriptions. A change described as "fixed auth bug" produces a generic bugfix note. "Fixed a race condition in the token refresh flow that caused intermittent 401s under high load" produces a specific, useful one.
- The tweet and email subject are for the overall release. If individual features warrant their own announcements, generate those separately.

---

## License

MIT.
