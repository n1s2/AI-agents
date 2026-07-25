# seo-content-brief-generator

Content briefs that just say "write 1500 words about project management software" produce generic content that ranks nowhere. A good brief tells the writer the exact search intent, a specific angle that differentiates from existing results, which questions must be answered, the precise outline with word counts per section, keyword placement guidance, and what to avoid. This searches the live SERP for the target keyword then generates a complete, actionable brief.

---

## What it does

Takes target keyword, secondary keywords, product name, content type, content goal, and target audience. Searches Tavily for current SERP results. Claude produces:

- **Title options** — 3 options, compelling, includes keyword, not clickbait
- **Meta description** — under 160 chars, keyword included, specific benefit
- **Search intent** — informational/navigational/transactional/commercial with explanation of what the searcher actually wants
- **Content angle** — the specific differentiation that makes this piece stand out from existing results
- **Key questions to answer** — specific questions the article must address to satisfy intent
- **Outline** — section-by-section with H2 headings, purpose, suggested word count, key points, and H3 subsections
- **Keyword targets** — each keyword with suggested placement and natural density
- **Differentiation from SERP** — specific ways this piece should differ from what already ranks
- **Internal link opportunities** — topics to link to
- **Content warnings** — what to avoid (outdated angles, overclaimed topics, common mistakes)
- **Writer notes** — context for the writer on what makes this brief non-generic

---

## Stack

n8n, Tavily API, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-content-brief \
  -H "Content-Type: application/json" \
  -d '{
    "target_keyword": "project management software for small teams",
    "product_name": "Flowdesk",
    "secondary_keywords": ["team task management", "simple project management", "ops team software"],
    "content_type": "blog_post",
    "content_goal": "organic_traffic",
    "word_count_target": 1800,
    "target_audience": "Operations managers at 15-60 person companies looking to replace spreadsheets",
    "brand_voice": "Direct and practical — we trust readers to make their own decisions",
    "cta": "Start free trial at flowdesk.com — no credit card required",
    "existing_content": "We already have articles on: team productivity tips, how to run a daily standup, Asana alternatives",
    "reply_email": "content@flowdesk.com"
  }'
```

**Required:** `target_keyword`, `product_name`

---

## Search intent accuracy

Claude distinguishes between what a keyword looks like and what searchers actually want. "Project management software for small teams" looks transactional but is mostly informational — people researching options, not ready to buy. The brief reflects this in tone, structure, and CTA placement (soft, not aggressive).

---

## SERP differentiation

After seeing what currently ranks, Claude identifies a specific angle that stands out — not just "write better content" but "the existing results all compare tools feature-by-feature; this piece should start from the job to be done and work backward to tool selection." Writers know exactly how to be different.

---

## Cannibalization protection

Pass existing content URLs or topics in `existing_content` and Claude will note where there's topical overlap and how to differentiate this piece from what you've already written.

---

## Limitations

- SERP data is from Tavily web search — it reflects publicly indexed results at the time of the call, not clickstream or ranking position data.
- This brief assumes the writer understands the product. For freelance writers, supplement with product context.

---

## License

MIT.
