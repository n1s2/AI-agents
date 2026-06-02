# grant-writing-assistant

Grant writing is slow, specific, and unforgiving. Funders read hundreds of applications. Ones that lead with the organization's history instead of the problem, use vague language like "comprehensive services," or don't explicitly mirror the funder's stated priorities get filtered out fast.

This takes your project brief and funder details, searches the web for the funder's current priorities and language, and writes a complete grant narrative: executive summary, statement of need with data, project description with a logic model, organizational capacity, evaluation plan, and budget narrative. It also flags weaknesses in the application before you submit, suggests specific strengthening actions, and pulls the funder's actual language to use throughout.

Works for foundation grants, federal grants, corporate giving programs, and community funds. Can generate a full narrative or a single section if that's what you need.

---

## What it does

1. Accepts a POST: org name and description, project title and description, funder name and priorities, grant type, amount requested, target population, expected outcomes, section needed
2. Searches Tavily for the funder's current priorities and grant language
3. Claude writes the grant narrative:
   - Alignment analysis — how this project matches the funder
   - Executive summary (150–200 words)
   - Statement of need with data and statistics
   - Project description with logic model narrative
   - Organizational capacity section
   - Evaluation plan with specific metrics
   - Budget narrative justifying the ask
4. Red flag check — weaknesses reviewers might push back on
5. Strengthening suggestions — specific improvements before submission
6. Funder language tags — exact phrases to use from their materials
7. Builds a clean grant document HTML
8. Emails if `reply_email` provided
9. Returns full JSON with all sections

---

## Stack

- **n8n** — webhook + workflow
- **Tavily API** — funder research
- **Anthropic Claude** (claude-sonnet-4-20250514) — grant writing
- **SMTP** — optional email delivery

---

## Setup

### 1. Environment variables

```
TAVILY_API_KEY=tvly-your-key
FROM_EMAIL=grants@yourorg.org
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** (optional)

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/write-grant \
  -H "Content-Type: application/json" \
  -d '{
    "org_name": "Eastside Youth Arts Collective",
    "org_description": "A nonprofit founded in 2015 serving youth ages 14-24 in the East Oakland flatlands. We provide free studio access, mentorship from professional artists, and paid apprenticeships. We have served 2,400 young people to date and maintain a 78% program completion rate.",
    "org_type": "nonprofit",
    "project_title": "Paid Creative Apprenticeship Expansion",
    "project_description": "We propose to expand our 12-week paid apprenticeship program from 24 slots to 60 slots annually, adding two new tracks in digital media and music production. Apprentices earn $18/hour while learning professional skills alongside working artists. Program graduates have gone on to full-time creative employment at a rate of 64%.",
    "funder_name": "Zellerbach Family Foundation",
    "funder_priorities": "Arts and culture, youth development, economic opportunity for low-income Bay Area residents",
    "grant_type": "foundation",
    "funding_amount_requested": 85000,
    "currency": "USD",
    "project_duration": "18 months",
    "target_population": "Low-income youth ages 16-22 in East Oakland, 80% BIPOC, 60% first-generation students",
    "expected_outcomes": "36 additional apprentices served annually, 85% program completion rate, 60% employment in creative sector within 6 months of graduation",
    "previous_funding": "Received $45,000 from Zellerbach in 2022 for our core studio operations program",
    "section": "full_narrative",
    "reply_email": "director@eastsideyoutharts.org"
  }'
```

**Required:** `org_name`, `project_title`, `funder_name`, `funding_amount_requested`, `project_description`

---

## Single section mode

Pass `section` to generate just one part of the application:

```json
"section": "statement_of_need"
```

Available sections: `executive_summary`, `statement_of_need`, `project_description`, `organizational_capacity`, `evaluation_plan`, `budget_narrative`, `full_narrative`

Useful when you already have most of the application written and need help with a specific section, or when the funder's portal has separate fields for each.

---

## Grant types

`federal`, `state`, `foundation`, `corporate`, `community`, `research`, `arts`, `education`, `health`, `other`

The type influences tone and structure — federal applications have different expectations than community foundation grants.

---

## The alignment analysis

The output opens with a 2–3 sentence analysis of how this specific project aligns with this specific funder's priorities. This is the lens through which the whole narrative is written — if the alignment is weak, the coaching flags it upfront.

---

## Funder language

One of the most common grant writing mistakes is using your own organization's language instead of the funder's. If a funder talks about "equitable access to creative economy careers" and you write about "job training," you're not speaking their language even if you're saying the same thing.

The `funder_language_to_use` section pulls specific phrases from the funder's materials (via Tavily research plus the priorities you provide) and presents them as tags to weave into the final application.

---

## Red flags

Claude reviews the application as submitted and flags weaknesses: vague impact claims, population sizes that seem unrealistic, outcomes that aren't measurable, budget amounts that need justification, or missing information reviewers typically expect. Fix these before submitting.

---

## The logic model

The project description section is written with a logic model narrative embedded: inputs (staff, funding, facilities) → activities (what you do) → outputs (units of service) → outcomes (changes in participants) → impact (broader change). Reviewers at foundations look for this structure explicitly.

---

## Previous funding

If you've received funding from this funder before, include it in `previous_funding`. Claude will reference the relationship and frame this as a continuation or expansion rather than a cold application — which significantly improves success rates with repeat funders.

---

## Without Tavily

Remove the **Research Funder** and **Merge Funder Context** nodes, connect **Valid?** directly to **Claude Grant Writer**, replace `{{ $json.funderContext }}` with an empty string. Claude uses its own knowledge of the funder. Works well for major foundations (Ford, Kellogg, MacArthur, Gates), less well for regional or newer funders. Supplement with the `funder_priorities` field.

---

## Limitations

- Claude doesn't know your organization's specific history, staff credentials, or program data. The more detail you put in `org_description`, `expected_outcomes`, and `project_description`, the stronger the output.
- Statistics in the statement of need are placeholders when not provided — tag them with [VERIFY] and replace with actual data before submitting.
- Grant writing requires iteration. The output is a strong first draft, not a submission-ready document. Budget time for revision, review by program staff, and legal/financial review of the budget narrative.

---

## Ideas

- [ ] Letter of inquiry generator: a shorter companion for LOI-first funders
- [ ] Reporting template: once funded, generate a mid-term or final report template based on the approved application
- [ ] Prospect research: given your project description, search for funders whose priorities align
- [ ] Application tracker: log submitted applications, deadlines, and outcomes to Google Sheets

---

## License

MIT.
