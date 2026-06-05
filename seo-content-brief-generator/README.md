# seo-content-brief-generator

Most content briefs are either a keyword and a word count, or a 20-page document nobody reads. Neither helps writers produce content that actually ranks.

This takes a target keyword, researches the current SERP and People Also Ask results via Tavily, and produces a complete content brief: search intent classification, a recommended title with alternatives, meta description, secondary keywords, a detailed section-by-section structure with notes per section, must-include topics, FAQ questions to answer, competitor analysis (what ranks, why, where the gaps are), E-E-A-T requirements, and technical SEO notes. It also gives a clear content angle — the specific hook that differentiates this piece from the existing top 10.

For content teams producing more than a few pieces per month, this replaces 30–60 minutes of manual keyword and SERP research per brief.

---

## What it does

1. Accepts a POST: target keyword, content type, business description, audience, related keywords, competitors, content goal, word count, tone
2. Runs two Tavily searches in parallel: SERP results for the keyword + People Also Ask research
3. Claude analyzes the competitive landscape and generates:
   - Search intent: type (informational/commercial/transactional/navigational) + stage (awareness/consideration/decision)
   - Recommended title with 2 alternatives
   - Meta description (150–160 chars)
   - Primary + 6–10 secondary keywords
   - A specific content angle that differentiates from what's ranking
   - Identified content gap in current top results
   - Section-by-section structure: heading, type, purpose, specific notes, word count
   - Must-include topics and statistics
   - 4–6 FAQ questions matching PAA queries
   - Competitor analysis: what ranks, weaknesses, opportunity
   - E-E-A-T requirements specific to this content
   - Technical SEO notes (schema, featured snippet opportunity)
4. Builds a color-coded HTML brief document
5. Emails if `reply_email` provided
6. Returns full JSON

---

## Stack

- **n8n** — webhook + workflow
- **Tavily API** — SERP + PAA research (two parallel searches)
- **Anthropic Claude** (claude-sonnet-4-20250514) — brief generation
- **SMTP** — optional email delivery

---

## Setup

### 1. Environment variables

```
TAVILY_API_KEY=tvly-your-key
FROM_EMAIL=seo@yourcompany.com
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** (optional)

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/seo-brief \
  -H "Content-Type: application/json" \
  -d '{
    "target_keyword": "project management software for small business",
    "content_type": "comparison_page",
    "business_description": "We sell Flowdesk, a lightweight project management tool designed for small teams under 20 people. Key differentiator: setup in 5 minutes, no training required.",
    "target_audience": "Small business owners and operations managers at 5-20 person companies who are overwhelmed by tools like Asana or Jira",
    "related_keywords": ["small business project tracker", "simple task management", "team task manager", "Trello alternative small business"],
    "competitors": ["Asana", "Monday.com", "Trello", "ClickUp", "Basecamp"],
    "content_goal": "organic traffic + free trial signups",
    "word_count_target": 2200,
    "tone": "direct and practical",
    "reply_email": "content@company.com"
  }'
```

**Required:** `target_keyword`, `content_type`

---

## Content types

| Type | Structure calibration |
|---|---|
| `blog_post` | Intro + body sections + conclusion, educational |
| `landing_page` | Benefits-led, CTA-focused, minimal FAQ |
| `product_page` | Features + specs + trust signals |
| `comparison_page` | Comparison table + per-competitor breakdown |
| `how_to_guide` | Numbered steps + tips + common mistakes |
| `listicle` | Numbered items with detailed explanations |
| `pillar_page` | Comprehensive hub with topic clusters |
| `case_study` | Challenge + solution + results structure |
| `faq_page` | Question-first structure throughout |

The structure section adapts to the content type — a comparison page gets a comparison table section and per-competitor sections; a how-to guide gets numbered steps.

---

## The content angle

This is the most important field in the brief and the one most tools skip. The `content_angle` isn't the topic — it's the specific hook or perspective that makes this piece worth reading when ten similar pieces already exist.

Example for "project management software for small business":
- Generic: "A guide to project management software"
- Angle: "Why most small businesses are using tools built for enterprises and paying the penalty in setup time and training costs"

Claude derives the angle from the SERP gap analysis — what the top results are missing or doing poorly.

---

## Secondary keywords

The brief includes 6–10 secondary keywords that should appear naturally in the content. These aren't stuffed — Claude selects semantically related terms based on SERP patterns and your related_keywords input. They help the piece rank for a broader keyword cluster.

---

## E-E-A-T requirements

For YMYL-adjacent topics or competitive SERPs, E-E-A-T (Experience, Expertise, Authoritativeness, Trustworthiness) signals increasingly influence rankings. The brief flags what this specific content needs — things like: "Author bio with verifiable credentials in project management", "Include customer data or internal case study results", "Cite G2 or Capterra reviews for third-party validation."

---

## Competitor analysis

Three fields:
- **What ranks** — the type of content dominating the SERP and why Google favors it
- **Weaknesses** — where current top results fall short (too generic, outdated, missing specific use case, bad UX)
- **Differentiation opportunity** — the specific gap to exploit

This is derived from actual SERP data for the keyword, not generic advice.

---

## Technical SEO notes

Includes:
- Whether a featured snippet opportunity exists and what format to target
- Schema markup recommendations (FAQ schema, HowTo schema, Article schema)
- Canonical strategy notes if there's potential for duplicate content

---

## Without Tavily

Remove the **SERP Research**, **PAA Research**, and **Merge SERP Data** nodes. Connect **Valid?** directly to **Claude Brief Writer** and replace `{{ $json.serpContext }}` and `{{ $json.paaContext }}` with empty strings. Claude uses its own knowledge of the keyword landscape — works well for evergreen topics, less precise for competitive niches where current SERP understanding matters.

---

## Limitations

- Tavily returns top web results but not Google's exact SERP (featured snippets, local packs, image carousels aren't directly visible). For precise SERP feature analysis, supplement with a proper SEO tool.
- Keyword volume and difficulty data aren't included — add an Ahrefs or SEMrush API call before the Claude node if you want those metrics in the brief.
- The brief is a starting point for a writer, not a guarantee of ranking. Content quality, site authority, and on-page optimization all affect actual ranking results.

---

## Ideas

- [ ] Bulk brief generation: submit a list of keywords from a Google Sheet, generate briefs in sequence
- [ ] Brief-to-content: chain this with a content writing agent that takes the brief as input and drafts the piece
- [ ] Competitive content gap finder: given your existing content, find keyword opportunities competitors rank for that you don't
- [ ] Brief approval workflow: save briefs to a Google Sheet for content manager review before assigning to writers

---

## License

MIT.
