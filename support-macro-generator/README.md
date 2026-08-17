# support-macro-generator

Support macros written once and never revisited become robotic — customers can tell when they're reading a canned response, and generic macros don't actually reflect how the issue should be explained. This generates a complete macro library from your list of common issues, with clear personalization variables and guidance on when each macro fits versus when it doesn't.

---

## What it does

Takes product name, brand voice, tone guidelines, and a list of common issues (each with summary, resolution steps, category, and frequency). Claude produces for each issue:

- **Macro name** — short internal reference name
- **Subject line** — if this is often the first response
- **Macro body** — complete text with `{{placeholder}}` variables clearly marked, covering greeting, acknowledgment, resolution steps or explanation, and a closing that invites follow-up
- **Placeholder variables** — list of variables used
- **When to use** — specific guidance on fit vs when it doesn't apply
- **Personalization note** — what the agent should customize beyond just filling placeholders, to avoid sounding robotic

Also produces general guidelines for the team on using macros well without losing the human touch.

HTML output shows each macro as a card with the templated body, placeholder chips, and usage guidance.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-support-macros \
  -H "Content-Type: application/json" \
  -d '{
    "product_name": "Flowdesk",
    "brand_voice": "Direct and warm — we treat customers like smart people who deserve real answers, not corporate hedging",
    "support_tone_guidelines": "Never say we apologize for any inconvenience. Get to the fix fast. Use first names. Sign off with the actual agent name, not a team signature.",
    "reply_email": "support-lead@flowdesk.com",
    "common_issues": [
      {"id": "ISS-001", "category": "billing", "frequency": "very_common", "issue_summary": "Customer was charged after cancelling their subscription", "resolution_steps": "Verify cancellation date in Stripe. If charge occurred after cancellation, issue full refund immediately and confirm cancellation is processed. If charge was for the period before cancellation, explain prorated billing."},
      {"id": "ISS-002", "category": "technical", "frequency": "common", "issue_summary": "Tasks not syncing to Notion integration", "resolution_steps": "Ask customer to check integration status in Settings > Integrations. Common fix: reconnect the integration (OAuth token often expires after 90 days). If reconnecting does not fix it, escalate to engineering with workspace ID."},
      {"id": "ISS-003", "category": "onboarding", "frequency": "common", "issue_summary": "New user cannot figure out how to invite team members", "resolution_steps": "Direct to Settings > Team Members > Invite. Note: only Workspace Admins can invite. If they are not an admin, they need to ask their admin or we can check who the admin is."},
      {"id": "ISS-004", "category": "billing", "frequency": "occasional", "issue_summary": "Customer wants to downgrade their plan but is confused about what happens to their data", "resolution_steps": "Explain that downgrading never deletes data — some features become read-only if over the new plan limits (e.g. extra users get view-only access). No data loss."}
    ]
  }'
```

**Required:** `common_issues`, `product_name`

---

## Not robotic

Every macro includes a `personalization_note` — specific guidance beyond just filling in `{{customer_name}}`. For the billing refund macro, this might be "acknowledge the specific frustration of being charged after cancelling before jumping to the fix" — a cue for the agent to add a sentence of genuine acknowledgment rather than pasting the macro verbatim.

---

## When to use / when not to use

Each macro includes explicit guidance on fit. The Notion sync macro, for example, would note it applies to the common OAuth expiration case but not to more complex sync issues that need engineering investigation — preventing agents from reaching for the macro when it doesn't actually apply.

---

## Placeholder variables

Variables are clearly marked with `{{double_brace}}` syntax and listed separately, making it easy to import into your actual helpdesk platform (Zendesk, Intercom, Help Scout) and map to their placeholder syntax.

---

## Limitations

- Macros are starting templates — review for accuracy against your actual product before deploying, especially resolution steps that reference specific UI paths.
- Up to 20 issues per call. For a larger macro library, batch by category (billing macros, technical macros, onboarding macros) across multiple calls.

---

## License

MIT.
