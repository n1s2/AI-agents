# vendor-evaluation-agent

Software vendor evaluations are one of those processes that either happens too informally ("we demoed it and it felt good") or too formally (a 40-page RFP process that costs more than the software). Most teams end up somewhere in the middle — a spreadsheet, some demos, a gut feeling.

This gives you a structured evaluation framework without the overhead. You define your criteria and weights, optionally pre-score what you already know, and the agent researches the vendor, assesses them against your criteria, compares to alternatives, flags red flags, gives you questions to ask in the demo, and tells you what to test in a proof of concept.

Works for SaaS tools, infrastructure vendors, professional services, and any other vendor category where you need to make a documented, defensible decision.

---

## What it does

1. Accepts a POST: vendor name, category, use case, evaluation criteria with weights and optional pre-scores, budget, alternatives to compare, integration and security requirements
2. Runs two parallel Tavily searches: vendor reviews/reputation + alternatives comparison
3. Claude evaluates:
   - Overall verdict: strong fit / good fit / acceptable / poor fit / not recommended
   - Confidence level
   - Executive summary (3–4 sentences, bottom-line recommendation)
   - Score suggestion per criterion with evidence from research
   - Strengths and weaknesses specific to the use case
   - Red flags (vendor lock-in, pricing opacity, acquisition instability, etc.)
   - Comparison to each alternative (when to choose this vendor vs each competitor)
   - Implementation and cost assessment (total cost including setup, training, overages)
   - Questions to ask in the demo (based on evaluation gaps)
   - What to test in a POC (concrete, specific)
4. Calculates weighted score if all criteria are pre-scored
5. Builds HTML evaluation report with score bars
6. Emails if `reply_email` provided
7. Returns full JSON

---

## Stack

- **n8n** — webhook + workflow
- **Tavily API** — vendor and comparison research (two parallel searches)
- **Anthropic Claude** (claude-sonnet-4-20250514) — evaluation analysis
- **SMTP** — optional email delivery

---

## Setup

### 1. Environment variables

```
TAVILY_API_KEY=tvly-your-key
FROM_EMAIL=procurement@yourcompany.com
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** (optional)

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/evaluate-vendor \
  -H "Content-Type: application/json" \
  -d '{
    "vendor_name": "Notion",
    "vendor_url": "https://notion.so",
    "category": "team wiki and knowledge management",
    "use_case": "Replace our current Confluence setup for a 45-person engineering and product team. Need to store technical documentation, product specs, meeting notes, and onboarding materials. Main pain points with Confluence: slow, hard to search, nobody keeps pages updated.",
    "evaluation_criteria": [
      { "name": "Search quality", "weight": 3 },
      { "name": "Ease of editing and maintenance", "weight": 3 },
      { "name": "Engineering workflow integration (GitHub, Jira)", "weight": 2 },
      { "name": "Migration from Confluence", "weight": 2, "score": 7, "notes": "Notion has an importer but we tested it — formatting is messy" },
      { "name": "Pricing at 45 seats", "weight": 2 },
      { "name": "Offline access", "weight": 1 }
    ],
    "budget": "$10-15 per user per month",
    "currency": "USD",
    "team_size": "45 people",
    "alternatives": ["Confluence", "Coda", "GitBook"],
    "integration_requirements": "Slack notifications, GitHub linking, Jira ticket embedding",
    "security_requirements": "SOC 2 Type II, SSO, admin controls for page permissions",
    "additional_context": "We tried Notion 2 years ago but adoption failed. Team leads were skeptical but agreed to re-evaluate given significant product improvements.",
    "reply_email": "ops@company.com"
  }'
```

**Required:** `vendor_name`, `category`, `use_case`, `evaluation_criteria` (minimum 2 items)

---

## Evaluation criteria format

Two formats work:

**Simple (name only):**
```json
"evaluation_criteria": ["Search quality", "Pricing", "Support quality"]
```

**Weighted with pre-scoring:**
```json
"evaluation_criteria": [
  { "name": "Search quality", "weight": 3 },
  { "name": "Pricing", "weight": 2, "score": 8, "notes": "Confirmed pricing in demo" },
  { "name": "Support quality", "weight": 1 }
]
```

Weights are relative — `weight: 3` means three times as important as `weight: 1`. They're normalized to percentages automatically.

If you provide `score` for all criteria (0–10), the agent calculates a weighted overall score. Partial pre-scoring is fine — Claude will suggest scores for the unscored criteria based on research.

---

## Weighted scoring

When all criteria have scores, the agent computes:
```
weighted_score = Σ(score × normalized_weight_%)
```

Shown prominently in the report. Useful for comparing multiple vendors evaluated against the same criteria — run this webhook once per vendor, then compare the weighted scores.

---

## Red flags

Claude flags specific concerns that warrant pause — not generic risks but things found in actual research or inferred from the vendor's situation:
- Pricing that changes significantly post-contract
- Recent acquisition or leadership change that signals instability
- Known issues with data portability or export
- Weak documentation that predicts support problems
- Excessive customer service complaints across review sites
- Features marketed as included that require upgrade

---

## Questions for the demo

The `questions_for_vendor` section is derived from gaps in the evaluation — things the research didn't answer clearly or where claims need verification. For the Notion example: "Can you show the admin controls for restricting sensitive pages to specific teams?" or "What does the Confluence migration process look like in practice — can we do a test migration with 10 pages?"

These are the questions that separate a real evaluation from a sales demo.

---

## POC criteria

If the evaluation warrants a proof of concept or trial, the `suggested_poc_criteria` gives specific things to test — not "evaluate usability" but "have 3 engineers migrate their current Confluence space and rate the quality of the import." Concrete and testable.

---

## Comparing multiple vendors

Run the webhook once per vendor with the same `evaluation_criteria` structure. Since weights and criteria are consistent, the weighted scores are directly comparable. Build a comparison table from the results.

---

## Without Tavily

Remove the **Research Vendor**, **Research Alternatives**, and **Merge Research** nodes. Connect **Valid?** directly to **Claude Vendor Evaluator** and replace `{{ $json.vendorContext }}` and `{{ $json.altContext }}` with empty strings. Claude uses its own knowledge — works well for well-known SaaS tools, less well for niche vendors or recent releases. Supplement with `additional_context` to provide what you know.

---

## Limitations

- Research is from indexed web content — G2 reviews, tech blog posts, comparison sites. It may miss very recent changes (pricing updates, new features, acquisition announcements from the last few weeks).
- Claude assesses based on available evidence plus your use case. For security-critical or regulated procurement, supplement with a formal security questionnaire and legal review.
- The weighted score reflects the criteria you defined. If you weight price heavily and the vendor is cheap, the score looks good even if support is weak. Define criteria that reflect what actually matters for your decision.

---

## Ideas

- [ ] Multi-vendor comparison: submit multiple vendors, generate a side-by-side comparison table
- [ ] G2/Capterra integration: pull verified review data directly before evaluation
- [ ] Evaluation history: log all evaluations to Google Sheets for institutional memory
- [ ] RFP generator: given the evaluation criteria, generate a structured RFP to send to vendors

---

## License

MIT.
