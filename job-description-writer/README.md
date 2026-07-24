# job-description-writer

Generic job descriptions attract generic candidates. "Passionate team player who thrives in a fast-paced environment" tells a strong candidate nothing about whether this role is right for them. This generates a complete, specific job description that leads with why the role is interesting, is honest about what the job actually involves (including its challenges), separates must-haves from nice-to-haves, and flags language that may unnecessarily filter candidates.

---

## What it does

Takes job title, company name, seniority level, role description, requirements, benefits, and hiring manager notes. Claude writes:

- **Headline** — 1–2 sentence hook explaining why this specific role at this specific company is interesting right now
- **About the role** — 2–3 paragraphs on what the person will do, what they own, and what success looks like in year one
- **About the team** — who they work with and how the team operates
- **What you'll do** — specific responsibilities starting with action verbs
- **Requirements** — must-have (non-negotiable, specific, justified) and nice-to-have separated
- **Honest about this role** — what the role is NOT, or what's genuinely hard about it
- **About the company** — mission, stage, what makes this interesting
- **Compensation & benefits** — in plain language
- **How to apply** — what the process looks like

Also returns: inclusion flags (phrases that may unnecessarily filter candidates), a 60-char posting headline for job boards, and a 300-char LinkedIn summary.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/write-job-description \
  -H "Content-Type: application/json" \
  -d '{
    "job_title": "Senior Backend Engineer",
    "company_name": "Flowdesk",
    "department": "Engineering",
    "seniority_level": "senior",
    "employment_type": "full_time",
    "location": "Remote, US",
    "remote_policy": "fully_remote",
    "company_stage": "Series A, 45 people",
    "salary_range": "USD 160,000–195,000 + equity",
    "company_mission": "Make operations teams as effective as engineering teams — give them the tools to run their work with the same rigor that engineers use to ship software.",
    "team_description": "4-person backend team. We own the API, integrations, and data pipeline. Direct access to customers. No bureaucracy — you will talk to users the week you join.",
    "role_description": "Own the integrations layer. We ship Notion sync this month and have 6 more integration partnerships lined up. You will design the architecture, write the code, and be the person customers call when something breaks.",
    "must_have_requirements": [
      "5+ years backend engineering, primarily Node.js",
      "Experience building and maintaining third-party integrations (webhooks, OAuth, REST)",
      "Comfortable owning a system end-to-end — architecture to production monitoring"
    ],
    "nice_to_have_requirements": [
      "Experience with Notion, Slack, or Linear APIs specifically",
      "Previous startup experience (we move fast and scope changes)"
    ],
    "benefits": ["Fully remote", "Health dental vision", "USD 1000 home office stipend", "Unlimited PTO", "Equity"],
    "hiring_manager_notes": "We need someone who is comfortable being the sole expert on integrations — there is no senior integrations engineer to ask. The right person finds that exciting not scary. We lost a candidate last round because they wanted more mentorship than we can provide at this stage.",
    "tone": "direct_and_human",
    "reply_email": "hiring@flowdesk.com"
  }'
```

**Required:** `job_title`, `company_name`

---

## Honest about the role

The `honest_about_the_role` field surfaces the genuine challenges or realities of this position. For the example above: "This is not a role with a dedicated QA team or a senior integrations lead to consult — you will be building the function from scratch." Strong candidates appreciate honesty; weak fits self-select out. Both outcomes are good.

---

## Inclusion flags

Claude reviews the generated JD and flags phrases that may unnecessarily filter candidates — things like "rockstar engineer," years-of-experience requirements that exceed the technology's age, cultural fit language that can screen on personality rather than competence, or degree requirements where they aren't justified. These appear highlighted at the bottom of the HTML doc.

---

## Requirements discipline

Claude distinguishes must-have from nice-to-have based on what you provide, and won't inflate requirements. If you pass "5+ years Node.js" as a must-have, it stays a must-have. If you don't specify, Claude derives requirements from the role but is instructed to avoid inflated experience requirements.

---

## Limitations

- JD is a draft — review for accuracy before posting, especially requirements and compensation.
- For roles where the JD will be legally reviewed (e.g., roles with EEO statements required), add the standard legal language post-generation. The inclusion flags are not legal advice.

---

## License

MIT.
