# pr-description-writer

PR descriptions are usually an afterthought — a one-liner or copy-paste of commit messages. Reviewers then have to reverse-engineer the intent, figure out how to test it, and guess what could break. This generates a complete, structured PR description from a summary of what changed: why it was built, what changed, how to test it, what reviewers should focus on, and what could go wrong.

---

## What it does

Takes PR title, changes summary, diff stats, linked tickets, testing notes, deployment notes, and reviewer focus areas. Claude writes a complete GitHub-flavored markdown PR description including:

- Summary (what and why, not a commit dump)
- Motivation/context (what problem this solves)
- Changes made (specific, structured)
- How to test (with checkbox steps)
- Screenshots placeholder if applicable
- Breaking changes section if flagged
- Deployment notes if provided
- Reviewer notes (where to focus, known tradeoffs)

Also returns: review checklist (specific things reviewers should check), risk assessment (low/medium/high) with rationale, and suggested GitHub labels.

The `markdown_description` field is ready to paste directly into GitHub.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/write-pr-description \
  -H "Content-Type: application/json" \
  -d '{
    "pr_title": "feat: add bidirectional Notion task sync",
    "pr_type": "feature",
    "breaking_change": false,
    "author_name": "Tom Walsh",
    "linked_tickets": ["FD-1201", "FD-1178"],
    "repo_context": "Flowdesk monorepo, Node.js backend, React frontend, Postgres",
    "testing_notes": "Integration tests in /tests/notion-sync. Requires NOTION_TEST_TOKEN env var. Manual testing: connect a real Notion workspace in staging, create a task in Flowdesk, verify it appears in Notion within 30s, and vice versa.",
    "deployment_notes": "Requires NOTION_CLIENT_ID and NOTION_CLIENT_SECRET env vars. No migrations needed. Feature-flagged behind ENABLE_NOTION_SYNC.",
    "reviewer_focus": "OAuth flow in notion-auth.ts, webhook deduplication logic in sync-handler.ts, and error handling for rate limit responses",
    "screenshots": true,
    "changes_summary": "Implemented Notion bidirectional sync via OAuth 2.0. Users connect their Notion workspace from Settings > Integrations. A webhook listener receives Notion page events and syncs to Flowdesk tasks. Flowdesk task mutations publish to a queue processed by a Notion updater worker. Handles deduplication via a sync_events table to prevent loops. Added connection management UI. Rate limiting: backs off on 429s with exponential retry."
  }'
```

**Required:** `pr_title`, `changes_summary`

---

## Markdown output

The `markdown_description` field is complete GitHub-flavored markdown. Copy it directly into the PR description field. Testing steps use GitHub checkbox syntax (`- [ ]`) which renders as interactive checkboxes in the PR.

---

## Review checklist

The `review_checklist` field gives reviewers a specific list of things to check — not generic advice but items derived from the changes. For the Notion sync example, it would include things like "verify webhook HMAC validation in notion-auth.ts" and "check that sync_events cleanup job is scheduled."

---

## Limitations

- Description quality depends on changes summary richness. A one-sentence summary produces a thin description. A detailed technical summary produces a thorough one.
- The agent writes the description from your summary — it doesn't read actual diff output. For fully automated PR descriptions from diffs, you'd need to pipe `git diff` output into `changes_summary`.

---

## License

MIT.
