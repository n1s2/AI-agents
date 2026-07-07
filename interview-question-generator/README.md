# interview-question-generator

Interview questions are often recycled or generic — "tell me about a challenge you overcame" asks the same thing regardless of role, seniority, or what you're actually trying to assess. This generates a structured interview guide for a specific role and interview type: questions mapped to competencies, follow-ups to probe depth, what good answers look like, and red flags to watch for.

---

## What it does

Takes job title, interview type, job description, required competencies, seniority level, and interview duration. Claude generates a complete guide:

- **Opening script** — how to start the interview and set expectations
- **Time allocation** — how to split the available time
- **Questions** — each with: question text, type (behavioral/technical/situational/hypothetical/motivational), difficulty (warmup/core/probe), competency tested, follow-up questions to go deeper, what a strong answer includes, and specific red flags
- **Scoring guide** — dimensions with 1/3/5 descriptors per dimension and weighting
- **Closing questions** — what to invite from the candidate
- **Debrief notes** — what interviewers should record immediately after

HTML output shows each question as a card with green "strong answer" and red "red flags" blocks side by side.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-interview-questions \
  -H "Content-Type: application/json" \
  -d '{
    "job_title": "Senior Product Manager",
    "interview_type": "behavioral",
    "seniority_level": "senior",
    "interview_duration_minutes": 60,
    "question_count": 8,
    "job_description": "Own the core workflow product, work directly with engineering, run weekly customer calls, accountable for feature adoption metrics.",
    "competencies_required": [
      "Product sense and customer empathy",
      "Cross-functional influence without authority",
      "Data-driven decision making",
      "Handling ambiguity and prioritization under pressure",
      "Communication with executives and engineers"
    ],
    "industry_context": "B2B SaaS, small team, no PM layer",
    "reply_email": "hiring@company.com"
  }'
```

**Required:** `job_title`, `interview_type`

---

## Interview types

`behavioral`, `technical`, `case_study`, `culture_fit`, `leadership`, `panel`, `phone_screen`

Each type gets different question formats. Behavioral uses STAR-format prompts. Technical probes specific skills. Case study uses scenario-based problems. Phone screen is lighter and focused on must-haves.

---

## Red flags vs strong answers

Every question includes both. Red flags are specific — not "seemed nervous" but "couldn't describe a specific example, only spoke in generalities" or "claimed full credit without mentioning the team." Strong answer criteria are also specific to what the role actually requires.

---

## Limitations

- This is a question guide, not an assessment platform. Scoring is manual — interviewers use the guide and record their own notes.
- For highly technical roles (staff engineer, data scientist), pass a detailed job description and set `interview_type` to `technical` — Claude can generate role-specific technical questions but needs context about the tech stack and seniority bar.

---

## License

MIT.
