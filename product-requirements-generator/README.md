# product-requirements-generator

PRDs written by PMs under time pressure tend to be solution specs: "build X with these fields." Engineering ends up asking why it was built, what problem it solved, and what "done" actually means. This generates a complete PRD from a problem statement outward — problem first, job stories second, solution approach third, then requirements and acceptance criteria.

---

## What it does

Takes feature name, problem statement, target users, job stories, proposed solution, success metrics, constraints, and out-of-scope items. Claude writes:

- TL;DR (1–2 sentence summary)
- Problem section (specific, evidence-based, not a feature pitch)
- Target users (primary and secondary)
- Job stories in "When [situation], I want to [motivation], so I can [outcome]" format
- Proposed solution (what the user experiences and why this approach — not implementation)
- Requirements split into must-have, should-have, nice-to-have — each with ID and rationale
- Acceptance criteria as Given/When/Then scenarios, edge cases flagged
- Out of scope list
- Success metrics table: metric / baseline / target / measurement method
- Open questions with owner and deadline
- Technical considerations (what engineering needs to factor in)
- Dependencies
- Review checklist for stakeholders before approving

HTML output formatted as a clean PRD document with serif type.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-prd \
  -H "Content-Type: application/json" \
  -d '{
    "feature_name": "Bulk task assignment",
    "product_name": "Flowdesk",
    "author_name": "Priya Sharma",
    "prd_audience": "engineering",
    "detail_level": "standard",
    "problem_statement": "Operations managers frequently need to reassign large numbers of tasks when a team member leaves, a project changes ownership, or work is rebalanced. Currently this requires opening each task individually, which takes 5-10 minutes for small sets and over an hour for large ones. Customer support has received 34 requests for bulk actions in the last 6 months, and it is in the top 3 most requested features in our NPS follow-up surveys.",
    "target_users": "Operations managers and team leads at companies with 10+ person ops teams who do weekly workload rebalancing",
    "job_stories": [
      "When a team member is out sick, I want to reassign all their tasks quickly, so I can keep work moving without manually touching 50 tasks",
      "When we restructure project ownership, I want to move tasks between owners in bulk, so I can complete the transition in under 5 minutes"
    ],
    "proposed_solution": "A checkbox-based selection mechanism on the task list that enables multi-select, then a dropdown action menu for bulk operations (assign, move to project, change status, archive). Start with assign only.",
    "success_metrics": [
      "Time to reassign 20+ tasks: target under 60 seconds, baseline ~15 minutes",
      "Feature adoption: 40% of eligible users (those with 20+ tasks) use bulk assign within 60 days of launch",
      "Support tickets about reassigning tasks: reduce by 60% within 90 days"
    ],
    "out_of_scope": [
      "Bulk editing task details (due dates, descriptions)",
      "Bulk operations in mobile app (desktop only for v1)",
      "Undo functionality for bulk operations"
    ],
    "constraints": "Must not break existing task list performance — currently loads in under 200ms for lists up to 500 tasks. No new backend APIs for v1; use existing task update endpoint.",
    "reply_email": "priya@flowdesk.com"
  }'
```

**Required:** `feature_name`, `problem_statement`

---

## Problem-first structure

Claude is instructed to explain the problem before the solution. The problem section includes specific evidence where derivable — user research, support ticket counts, NPS mentions, time measurements. "Users have trouble with X" is not a problem statement; "Operations managers spend 15+ minutes reassigning tasks when team composition changes, which they do monthly" is.

---

## Requirements vs acceptance criteria

Requirements describe what the feature must do. Acceptance criteria describe how you know it's done — as testable Given/When/Then scenarios. Edge cases are flagged separately. Both are generated from the same input.

---

## Open questions

Claude identifies questions that should be answered before engineering starts — things the PRD doesn't resolve. Each question gets an owner role and needed-by date.

---

## Limitations

- PRD quality is proportional to problem statement quality. A one-line problem statement produces a thin PRD. A specific, evidence-backed problem statement with user context produces a strong one.
- Technical considerations are based on what the PM provides in `constraints` — Claude doesn't have access to your actual codebase or architecture.

---

## License

MIT.
