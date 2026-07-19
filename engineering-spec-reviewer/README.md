# engineering-spec-reviewer

Engineering specs that skip review often surface problems during implementation — missing error handling, unvalidated assumptions, scalability issues that weren't considered upfront. By then, rework is expensive. This reviews a spec as a senior engineer would: identifying critical gaps, flagging assumptions that need validation, asking the questions that will unblock implementation, and generating a pre-implementation checklist.

---

## What it does

Takes a spec document (technical design, API design, architecture, database schema, security review, migration plan, etc.), spec type, system context, and team standards. Claude reviews and returns:

- Overall verdict: approve / approve_with_changes / needs_revision / reject
- Quality score (1–10)
- 2–3 sentence summary of the overall assessment
- **Critical issues** — each with why it matters, a specific fix suggestion, and which section it relates to
- **Warnings** — less critical concerns worth addressing
- **Strengths** — specific things the spec does well
- **Missing sections** — what's absent that should be there
- **Assumptions to validate** — what the spec assumes without proving
- **Questions for author** — specific questions that need answers before approval
- **Implementation risks** — what the implementing engineer should watch out for
- **Pre-implementation checklist** — items to verify before starting

HTML output with critical issues prominently colored, checklist with interactive checkboxes, and verdict badge with score.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/review-eng-spec \
  -H "Content-Type: application/json" \
  -d '{
    "spec_title": "Notion Bidirectional Sync — Technical Design",
    "spec_type": "technical_design",
    "author_name": "Tom Walsh",
    "reviewer_context": "Staff engineer review, focus on reliability and edge cases",
    "system_context": "Node.js monorepo, PostgreSQL, Redis for queuing, deployed on AWS ECS. We process ~50k task mutations/day currently.",
    "team_standards": "All external integrations must have idempotency. Error states must be logged to Datadog. No synchronous calls to external APIs in request path.",
    "focus_areas": ["error handling", "race conditions", "scalability"],
    "reply_email": "tom@flowdesk.com",
    "spec_content": "[Paste the full spec document here — minimum 100 chars, up to 8000]"
  }'
```

**Required:** `spec_title`, `spec_content`

---

## Spec types

`technical_design`, `api_design`, `architecture`, `database_schema`, `security_review`, `performance`, `migration`, `integration`

Type affects what Claude looks for. Database schema reviews check for missing indexes, cascade behaviors, and normalization issues. Security reviews check for auth gaps, input validation, and data exposure. API design reviews check for consistency, versioning, and error response shapes.

---

## Team standards

Pass your actual team standards in `team_standards` — things like "all external API calls must have circuit breakers" or "database migrations must be backward compatible for one deploy cycle." Claude checks the spec against these specifically, not just generic best practices.

---

## Reviewer context

Pass what role the reviewer is playing in `reviewer_context`. "Staff engineer review focusing on scalability" produces different feedback than "junior engineer self-review checklist" or "security-focused review before SOC 2 audit." Claude calibrates depth and focus accordingly.

---

## Limitations

- Claude reviews the spec as written, not the actual implementation. Issues that only surface in code (subtle race conditions, performance under specific load patterns) won't be caught here.
- For security-critical specs, this is a first pass — not a replacement for a dedicated security engineer review.
- Spec quality must be high enough to review. A 3-sentence spec produces a thin review. A detailed design doc produces useful feedback.

---

## License

MIT.
