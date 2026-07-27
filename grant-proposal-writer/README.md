# grant-proposal-writer

Grant proposals written by committee tend to sound like it — passive voice, disconnected sections, generic impact claims. Proposals that win funding open with the problem in human terms, connect the work to the funder's specific priorities, demonstrate credibility without boasting, and have measurable outcomes that reviewers can evaluate. This generates a complete proposal draft with funder alignment notes and strengthening suggestions.

---

## What it does

Takes organization name, project title, funder, amount, problem statement, proposed solution, target population, organization background, and measurable outcomes. Claude writes:

- **Executive summary** — 150–200 words for a reviewer reading 50 proposals a week
- **Statement of need** — 2–3 paragraphs: problem in human terms, specific data, why now, who is affected
- **Project description** — 3–4 paragraphs: what will happen, how, by whom, when — activities not just hoped outcomes
- **Organizational capacity** — why this org is positioned to do this work, track record, community relationships
- **Project team** — key members and qualifications
- **Goals and objectives** — each goal with specific objectives, measurement method, target number, and timeline
- **Evaluation plan** — how success will be measured and reported
- **Sustainability plan** — how impact continues after the grant
- **Budget narrative** — plain-language explanation of how funds will be used
- **Timeline summary** — key activities by quarter or month
- **Closing statement**

Also returns: funder alignment notes (what to emphasize in the cover letter based on the funder's stated priorities), and strengthening suggestions (specific things that would make this proposal stronger if the information is available).

HTML formatted as a grant document with serif type.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/write-grant-proposal \
  -H "Content-Type: application/json" \
  -d '{
    "organization_name": "Eastside Workforce Collaborative",
    "project_title": "Bridging the Digital Skills Gap: Workforce Training for East Oakland Adults",
    "grant_funder": "California Workforce Development Board",
    "grant_amount": 175000,
    "currency": "USD",
    "geographic_focus": "East Oakland, Alameda County, California",
    "target_population": "Adults 25-54 without post-secondary credentials, unemployed or underemployed, in East Oakland zip codes 94601-94606",
    "funder_priorities": "Priority areas: closing skills gaps in technology and healthcare sectors; serving historically underserved communities; programs with clear employment outcomes; regional employer partnerships",
    "problem_statement": "East Oakland has 23% unemployment in the 25-54 age group compared to 8% countywide. 68% of residents lack a post-secondary credential. The fastest-growing local employers — tech, healthcare administration, logistics — require digital literacy skills that most adult residents have not had access to learn.",
    "proposed_solution": "12-week intensive digital skills training cohorts, 3 cohorts per year, 20 participants each. Curriculum co-designed with 4 regional employers who have committed to interview graduates. Includes childcare, transportation stipends, and wraparound support.",
    "organization_background": "Founded 2018. Served 340 adult learners. 78% job placement rate within 6 months. Staff of 8, all East Oakland residents. Partners include Oakland Unified Adult Education, Laney College, and 12 regional employers.",
    "measurable_outcomes": [
      "60 participants complete training annually (3 cohorts x 20)",
      "85% completion rate",
      "70% employed within 6 months",
      "Average wage of $22+/hour for placed graduates",
      "4 employer partners participating in curriculum design and hiring"
    ],
    "project_timeline": "Q1: Cohort 1 recruitment and employer curriculum sessions. Q2: Cohort 1 training. Q3: Cohort 2. Q4: Cohort 3 + year-end employer hiring fair.",
    "team_background": "Executive Director: 12 years workforce development. Training Coordinator: former community college instructor. Employer Relations Manager: 8 years regional economic development.",
    "budget_summary": "Personnel (60%): 2 FTE instructors, 0.5 FTE coordinator. Participant support (20%): childcare, transit stipends. Curriculum and materials (10%). Evaluation and reporting (10%).",
    "sustainability": "Employer partners have committed to continued curriculum partnerships. Year 2 funding strategy includes county workforce funds and employer training subsidies. Social enterprise model under development.",
    "reply_email": "grants@eastsideworkforce.org"
  }'
```

**Required:** `organization_name`, `project_title`, `grant_funder`

---

## Funder alignment

Pass the funder's stated priorities in `funder_priorities` (from their website, RFP, or past grants). Claude writes the proposal with those priorities in mind and includes a separate `funder_alignment_notes` section explaining exactly which elements to emphasize in the cover letter and where the proposal most directly addresses their priorities.

---

## Strengthening suggestions

Claude identifies specific things that would make the proposal stronger — data points to find, partnerships to mention, outcome metrics to add — based on gaps in the provided information. These are honest gaps, not boilerplate suggestions.

---

## Limitations

- Proposal is a first draft. Review for accuracy, especially outcome numbers, organizational statistics, and budget figures.
- Funder-specific formatting requirements (page limits, specific sections, electronic portals) vary. This generates content; adapt structure to the RFP.
- Not a substitute for understanding the specific funder — the better the `funder_priorities` input, the more aligned the proposal.

---

## License

MIT.
