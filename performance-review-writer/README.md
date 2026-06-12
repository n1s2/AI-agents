# performance-review-writer

Performance reviews take managers an average of 3–5 hours each. Most of that time isn't thinking about the employee — it's translating rough notes and impressions into coherent prose that's professional, specific, and won't create HR problems.

This takes your raw notes, bullet points, accomplishments, and development areas and produces a complete, section-structured performance review narrative. The output uses specific examples (not generic praise), balances strengths with constructive development feedback, and sets meaningful goals for the next period. The tone matches the rating — a high performer's review reads differently from a PIP conversation.

It also gives the manager internal delivery notes: how to handle a sensitive topic, what to emphasize in the conversation. Those stay off the review document.

---

## What it does

1. Accepts a POST: employee name, role, review period, type, manager's raw notes, accomplishments, development areas, goals, overall rating, competencies to cover
2. Claude writes a structured performance review:
   - Opening summary (2–3 sentences setting the overall tone)
   - Sections with narratives (Performance & Accomplishments, Collaboration, Development, etc. — calibrated to the review type)
   - Optional per-section ratings
   - Closing statement (forward-looking, appropriate to the rating)
   - Goals for next period (specific and measurable, with rationale)
   - Key strengths and development areas (summary view)
3. Returns a `full_review_text` field ready to paste into any HR system
4. Includes manager delivery notes (internal, not part of the employee-facing review)
5. Builds HTML formatted preview
6. Emails if `reply_email` provided

---

## Stack

- **n8n** — webhook + workflow
- **Anthropic Claude** (claude-sonnet-4-20250514) — review writing
- **SMTP** — optional email delivery

---

## Setup

### 1. Environment variables

```
FROM_EMAIL=hr@yourcompany.com
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** (optional)

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/write-review \
  -H "Content-Type: application/json" \
  -d '{
    "employee_name": "Jordan Park",
    "employee_title": "Senior Software Engineer",
    "role": "Senior Software Engineer",
    "department": "Platform Engineering",
    "review_period": "H1 2025",
    "review_type": "mid_year",
    "manager_name": "Priya Sharma",
    "overall_rating": "exceeds_expectations",
    "performance_notes": "Jordan has had a strong first half. Led the database migration project that we kept delaying — took ownership without being asked, coordinated across 4 teams, delivered 2 weeks ahead of schedule. Code review quality has been exceptional — Ive noticed other engineers referencing Jordans comments as the standard. Main gap: still reluctant to speak up in cross-functional meetings, tends to let others drive even when Jordan has the most relevant context. Had a good 1:1 in April where Jordan acknowledged this and seems to be working on it but progress has been slow.",
    "accomplishments": [
      "Led database migration ahead of schedule with zero production incidents",
      "Mentored 2 junior engineers — both have received positive peer feedback",
      "Reduced CI/CD pipeline run time by 40% through parallelization work"
    ],
    "areas_for_improvement": [
      "Cross-functional communication — speaking up more proactively in mixed-audience meetings",
      "Documentation — technical docs are thorough but shipped late for 3 of 5 major features"
    ],
    "goals_for_next_period": [
      "Lead the architecture review for the new event streaming system (H2 priority)",
      "Present at least 2 cross-functional syncs on Platform work",
      "Documentation shipped within 1 week of feature release for all H2 work"
    ],
    "competencies": ["Technical excellence", "Collaboration", "Communication", "Ownership"],
    "tone": "constructive",
    "word_target": 450,
    "include_section_ratings": false,
    "reply_email": "priya@company.com"
  }'
```

**Required:** `employee_name`, `review_period`, `role`, `performance_notes`

---

## Review types

| Type | Structure |
|---|---|
| `annual` | Comprehensive — full review of the year across all dimensions |
| `mid_year` | Focused check-in — progress update, not full retrospective |
| `quarterly` | Brief pulse on key areas and near-term goals |
| `probation` | End-of-probation assessment — fit, growth, next steps |
| `90_day` | New hire ramp — onboarding, early contributions, expectations |
| `pip` | Performance Improvement Plan structure — clear expectations, support, timeline |

The review structure and tone adapt significantly by type. A PIP has very different requirements from an annual review of a high performer.

---

## Rating scale

| Rating | Label |
|---|---|
| `exceptional` | Exceptional |
| `exceeds_expectations` | Exceeds Expectations |
| `meets_expectations` | Meets Expectations |
| `below_expectations` | Below Expectations |
| `unsatisfactory` | Unsatisfactory |

When `overall_rating` is provided, it colors the framing throughout the review — the language, tone, and balance of positive vs development feedback all shift.

---

## Competencies

The `competencies` array tells Claude which dimensions to cover. If not provided, Claude infers from the role. Common sets:
- Engineering: `Technical excellence, Collaboration, Communication, Ownership, Learning`
- Management: `People development, Strategic thinking, Execution, Communication, Culture`
- Sales: `Results, Customer focus, Product knowledge, Pipeline management, Collaboration`

---

## The manager delivery notes

Every review includes a `manager_notes` field — internal guidance not included in the employee-facing document. Examples:
- "Jordan has mentioned feeling underutilized. Lead with the database migration success before introducing the communication development area."
- "The documentation issue has been discussed before — frame this as a pattern to address rather than a new observation."
- "Strong review overall. Good opportunity to discuss Jordan's interest in a staff engineer path."

These don't go into the HR system — they're for the manager's prep.

---

## The `full_review_text` field

The JSON response includes a `full_review_text` field with the complete review as a formatted string with headers and paragraph breaks. Copy and paste into Workday, BambooHR, Lattice, or whatever HR system you use.

---

## Specificity in the notes

Claude can only write specifically if you give it specific material. "Jordan had a good year" produces vague output. "Jordan led the database migration that we'd delayed twice, took ownership without being asked, and delivered 2 weeks early with zero production incidents" produces a specific narrative.

The `performance_notes` field is free-form — write whatever you have in whatever order. Claude organizes it.

---

## Limitations

- Claude generates a strong first draft. Review it before submitting to your HR system — your knowledge of the employee and situation is irreplaceable.
- PIP reviews in particular have legal implications. Have HR review PIP drafts before delivering them to employees.
- The review reflects the notes you provide. If your notes are biased (halo effect, recency bias), the review will reflect that. The tool structures and polishes your notes; it doesn't correct for systematic performance assessment errors.

---

## Ideas

- [ ] Peer review synthesis: accept multiple peer feedback inputs, synthesize into a single assessment to incorporate into the manager review
- [ ] Self-review draft: companion endpoint where employees draft their own self-review from their own notes
- [ ] Goal tracking: log approved goals to a Google Sheet and surface them at the next review cycle
- [ ] Calibration prep: generate a one-paragraph summary per employee for calibration sessions

---

## License

MIT.
