# content-audit-analyzer

A content audit spreadsheet with 200 rows sorted by pageviews doesn't tell you what to do. Should this underperforming article be updated, merged with a similar one, redirected, or deleted? Is this cluster of posts cannibalizing each other? Are your top performers being leveraged or just sitting there? This analyzes your content inventory and produces specific, prioritized recommendations: keep, update, consolidate, redirect, delete, or repurpose — with rationale for each.

---

## What it does

Takes up to 80 content items with available performance data (pageviews, organic traffic, SERP position, conversion rate, backlinks, time on page, bounce rate, publish date, last updated). Claude produces:

- **Audit summary** — overall portfolio health, biggest opportunities, biggest dead weight
- **Content recommendations** — each piece with: action (keep/update/consolidate/redirect/delete/repurpose), priority, specific rationale based on the data, what exactly to do, and expected impact
- **Keyword cannibalization groups** — clusters of content competing for the same keyword, with recommendation on which to keep and what to do with the others
- **Top performers** — what's working well and how to leverage it further
- **Quick wins** — high-impact, low-effort actions with expected impact
- **Pruning candidates** — content to cut with specific reason
- **Content gaps** — topics or keyword clusters missing from the portfolio
- **Portfolio health** — strengths, systemic weaknesses, content diversity assessment

HTML report with action summary badges, full recommendation table, cannibalization cards, and pruning list.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/analyze-content-audit \
  -H "Content-Type: application/json" \
  -d '{
    "site_name": "Flowdesk Blog",
    "site_description": "B2B SaaS blog targeting operations managers. Goal: organic traffic and trial signups.",
    "content_goals": "Drive organic traffic from ops-focused keywords, convert to trial signups",
    "reply_email": "content@flowdesk.com",
    "content_items": [
      {"id": "POST-001", "url": "/blog/project-management-ops-teams", "title": "Project Management for Ops Teams", "type": "blog_post", "word_count": 2100, "publish_date": "2024-03-15", "last_updated": "2024-03-15", "pageviews_last90": 4200, "organic_traffic": 3800, "serp_position": 4.2, "target_keyword": "project management ops teams", "avg_time_on_page": 210, "bounce_rate": 0.58, "conversion_rate": 0.8, "backlinks": 12},
      {"id": "POST-002", "url": "/blog/ops-team-project-management-guide", "title": "The Complete Guide to Ops Team Project Management", "type": "blog_post", "word_count": 3800, "publish_date": "2024-09-01", "last_updated": "2024-09-01", "pageviews_last90": 180, "organic_traffic": 95, "serp_position": 28, "target_keyword": "ops team project management", "avg_time_on_page": 90, "bounce_rate": 0.82, "conversion_rate": 0.1, "backlinks": 1},
      {"id": "POST-003", "url": "/blog/asana-vs-spreadsheets", "title": "Asana vs Spreadsheets for Small Teams", "type": "blog_post", "word_count": 1400, "publish_date": "2023-11-20", "last_updated": "2023-11-20", "pageviews_last90": 890, "organic_traffic": 760, "serp_position": 8.5, "target_keyword": "asana vs spreadsheets", "conversion_rate": 1.4, "backlinks": 6},
      {"id": "POST-004", "url": "/blog/monday-vs-spreadsheets", "title": "Monday.com vs Spreadsheets: Which Is Better?", "type": "blog_post", "word_count": 1200, "publish_date": "2024-01-10", "pageviews_last90": 340, "organic_traffic": 290, "serp_position": 14, "target_keyword": "monday.com vs spreadsheets", "conversion_rate": 0.6, "backlinks": 2}
    ]
  }'
```

**Required:** `content_items`

---

## Action definitions

| Action | When to use |
|---|---|
| `keep` | Performing well, no action needed |
| `update` | Good topic/backlinks but stale content, declining rankings |
| `consolidate` | Multiple thin pieces on same topic — merge into one stronger piece |
| `redirect` | Low value but has backlinks — redirect to stronger related piece |
| `delete` | No traffic, no backlinks, no conversion value, not worth updating |
| `repurpose` | Good information but wrong format — could work better as video, guide, etc. |

---

## Cannibalization detection

Claude identifies clusters of content competing for the same or similar keywords. This is one of the most common causes of ranking stagnation — two posts splitting authority instead of one post dominating. The recommendation specifies which piece to keep and what to do with the others (usually consolidate or redirect).

---

## Pass whatever data you have

Not every field is required. Claude works with whatever subset is available — organic traffic, pageviews, SERP position, conversion rate, backlinks, publish date. More data = better recommendations. The minimum useful set is title, pageviews/traffic, and publish date.

---

## Limitations

- Up to 80 content items per call. For larger sites, run by content type (all blog posts, then all landing pages) or by topic cluster.
- Claude can't access your actual URLs or content directly. Pass the data from your analytics platform (GA4, Ahrefs, SEMrush) as the inventory.

---

## License

MIT.
