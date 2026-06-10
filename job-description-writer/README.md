# job-description-writer

Most job descriptions are terrible for the same reasons: they list 15 requirements when 6 are real, use gendered language that filters out qualified candidates, describe the company in marketing copy that tells the candidate nothing, and never say what makes the role actually interesting.

This generates job descriptions that attract strong candidates and filter out poor fits. The "about the role" section describes what the person will actually do, not a list of responsibilities copied from a template. Requirements are separated into must-haves and nice-to-haves honestly. There's an "honest about the challenges" section because candidates who join knowing the hard parts are more likely to stay.

Includes a bias flag check when enabled — looks for masculine-coded language, inflated degree requirements, and experience requirements that don't match the seniority level.

---

## What it does

1. Accepts a POST: job title, department, seniority level, work type, location, company description, team context, responsibilities, must-have and nice-to-have skills, salary range, benefits
2. Claude writes a complete JD:
   - Display title (clean, searchable, no internal codes)
   - Headline hook (what makes this role genuinely interesting)
   - About the company (2–3 specific sentences, not marketing copy)
   - About the role (what they'll actually do, what success looks like at 6 months)
   - What you'll do (5–8 concrete responsibilities with action verbs)
   - What we're looking for (5–7 honest must-haves, no inflation)
   - Bonus points (3–5 genuine nice-to-haves)
   - Honest about the challenges (1–2 sentences on what's genuinely hard)
   - What we offer (salary if provided + meaningful benefits)
3. If `avoid_biased_language` is true: flags any problematic language in the output
4. Provides interview signal: the one most important thing to assess in first-round interviews
5. Notes on job board searchability
6. Returns the full JD as a paste-ready text string
7. Builds HTML formatted preview
8. Emails if `reply_email` provided

---

## Stack

- **n8n** — webhook + workflow
- **Anthropic Claude** (claude-sonnet-4-20250514) — JD writing
- **SMTP** — optional email delivery

---

## Setup

### 1. Environment variables

```
FROM_EMAIL=hiring@yourcompany.com
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** (optional)

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/write-jd \
  -H "Content-Type: application/json" \
  -d '{
    "job_title": "Senior Product Designer",
    "department": "Product",
    "seniority_level": "senior",
    "work_type": "hybrid",
    "location": "New York, NY (2 days/week in office)",
    "company_stage": "Series B, 120 people",
    "company_description": "Meridian builds financial planning software for independent financial advisors. 4,000 advisors use us to manage $18B in client assets. We are growing 80% year over year and expanding from solo advisors to small RIA firms.",
    "team_context": "Reporting to the Head of Product. Working closely with 2 other designers and a 12-person product engineering team. This is the first senior design hire — youll help establish design systems and processes.",
    "core_responsibilities": [
      "Own end-to-end design for our core financial planning and client reporting workflows",
      "Build and maintain a design system from a partial foundation",
      "Run user research with advisors to identify workflow friction",
      "Partner with engineering to ensure design quality through implementation",
      "Mentor 2 junior designers as the team grows"
    ],
    "must_have_skills": ["5+ years product design", "Complex data visualization", "Figma", "User research", "Design systems"],
    "nice_to_have_skills": ["Fintech or regulated industry experience", "Motion design", "Familiarity with financial planning concepts"],
    "salary_range": "$145,000 - $175,000",
    "currency": "USD",
    "benefits": ["Equity", "Full health dental vision", "20 days PTO", "401k with 4% match", "$2,000 annual learning budget", "Home office stipend"],
    "avoid_biased_language": true,
    "tone": "professional",
    "reply_email": "hiring@meridian.co"
  }'
```

**Required:** `job_title`, `department`, `company_description`

---

## Seniority levels

`intern`, `junior`, `mid`, `senior`, `staff`, `principal`, `manager`, `director`, `vp`, `c_level`

Calibrates the scope of responsibilities, years of experience implied, and the tone of requirements. A senior role gets different language from a junior one even with identical responsibilities.

---

## The "honest about the challenges" section

This is the field most hiring managers want to skip and the one that makes the most difference. Candidates who accept a role knowing the hard parts stick around longer. Candidates who discover the hard parts after joining leave.

Examples of what this might say:
- "This is our first design hire — you'll be building process from scratch rather than inheriting a mature system."
- "Our engineering team is strong but design-to-implementation fidelity has been inconsistent. Part of this role is changing that."
- "We're in a high-growth moment, which means priorities shift quickly and you'll need to be comfortable with ambiguity."

---

## Bias flag check

When `avoid_biased_language` is true, Claude reviews the output for:
- Masculine-coded terms (rockstar, ninja, guru, crushing it, aggressive growth)
- Gendered pronouns
- Unnecessarily exclusive degree requirements ("must have a CS degree" when experience would suffice)
- Experience inflation ("7+ years required" for a mid-level role)
- Ageist language
- Terms that signal culture fit over competence

The flags appear in a "review before posting" section. If the JD is clean, the array is empty.

---

## The `full_jd_text` field

The JSON response includes a `full_jd_text` field — the complete job description as a formatted string with section headers and line breaks, ready to paste directly into Greenhouse, Lever, LinkedIn, or any other ATS.

---

## Interview signal

Every JD output includes a single most important thing to assess in first-round interviews — derived from the responsibilities and requirements in the JD. For a senior design role at a financial software company, it might be: "Assess ability to make complex financial data legible to non-experts — ask them to walk through a past example of simplifying a data-heavy workflow."

---

## Limitations

- The JD is based entirely on what you provide. If your `company_description` is vague ("we're building the future of X"), the "about the company" section will be vague. Specific input produces specific output.
- Bias flags are the model's assessment, not a legal review. For legally sensitive hiring situations, have HR review the JD regardless of the flag output.
- The word count isn't controlled. If you need a JD under a specific length for an ATS with limits, add a `max_words` field and instruct Claude in the prompt to keep within it.

---

## Ideas

- [ ] ATS integration: push the generated JD directly to Greenhouse, Lever, or Workday via their APIs
- [ ] Interview guide generator: companion agent that takes the JD and generates an interview scorecard with evaluation criteria per requirement
- [ ] JD library: save all generated JDs to Google Sheets, tag by department and date, build a reusable template library
- [ ] Candidate screening: given a JD and a resume, score the fit against the must-have requirements

---

## License

MIT.
