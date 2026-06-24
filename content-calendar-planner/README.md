# content-calendar-planner

Content planning usually means someone staring at a blank spreadsheet trying to think of posts. The mechanical work — figuring out pillar balance, channel-specific formats, frequency, trend angles — doesn't require human creativity. The hooks and ideas do. This handles the structure so you can focus on the execution.

Takes your brand, audience, channels, content pillars, and planning period. Searches for current industry trends via Tavily. Claude builds a week-by-week content calendar with specific post ideas — each one with a concrete hook, the content it covers, the format it should take, a CTA, and production notes. Not themes; actual post ideas a writer or designer can brief from.

---

## What it does

1. Takes brand info, target audience, channels with post frequencies, content pillars, planning period, upcoming events, tone
2. Searches Tavily for current content marketing trends in the industry
3. Claude builds:
   - Calendar overview and strategy for the period
   - Content pillars (with descriptions)
   - Week-by-week post ideas, each with: channel, pillar, format, hook, content idea, CTA, notes
   - Content mix rationale (why this balance suits this brand)
   - Topics/formats to avoid
   - Repurposing suggestions (turning planned content into additional assets)
4. Builds a color-coded HTML calendar (each channel has its own color)
5. Emails if `reply_email` provided
6. Returns full JSON

---

## Stack

n8n, Tavily API, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Setup

**Env vars:** `TAVILY_API_KEY`, `FROM_EMAIL`

**Credentials:** Anthropic API (LangChain node), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/plan-content-calendar \
  -H "Content-Type: application/json" \
  -d '{
    "brand_name": "Flowdesk",
    "brand_description": "Lightweight project management for small teams. Setup in 5 minutes, no training required.",
    "target_audience": "Operations managers and small business owners at 5-25 person companies frustrated with Asana and Jira",
    "industry": "B2B SaaS productivity tools",
    "channels": ["linkedin", "twitter", "blog"],
    "post_frequency": { "linkedin": 4, "twitter": 10, "blog": 2 },
    "content_pillars": ["productivity tips", "product updates", "customer stories", "industry insights"],
    "planning_period": "June 2025",
    "tone": "direct, practical, slightly irreverent",
    "upcoming_events": ["Summer pricing campaign June 15", "New Slack integration launch June 22"],
    "competitors": ["Asana", "Monday.com", "Trello"],
    "reply_email": "marketing@flowdesk.com"
  }'
```

**Required:** `brand_name`, `target_audience`, `channels`, `planning_period`

---

## Post ideas vs post templates

Each entry in `posts` is a specific idea, not a fill-in-the-blank template. The `hook` is the actual opening line or concept. The `content_idea` describes what the post covers specifically enough to brief someone else. You may edit them — but they're starting points, not prompts.

---

## Channel-specific formats

Claude knows that a LinkedIn post reads differently from a Twitter thread from a blog post. Post ideas are calibrated to the platform — LinkedIn gets professional insight framing, Twitter gets punchy takes or threads, Instagram gets visual-first concepts, blog posts get structured how-to or analysis angles.

---

## Pillar balance

Claude distributes posts across your content pillars throughout the period — you won't get five educational posts in week one and five promotional posts in week four. The `content_mix_rationale` explains the balance chosen and why it fits the brand and period.

---

## Repurposing suggestions

The calendar includes 2–3 suggestions for turning planned content into additional assets. A blog post becomes a LinkedIn carousel. A customer story becomes an email. A thread becomes a summary post. Gets more mileage from the same ideas.

---

## Limitations

- The calendar is a plan, not a guarantee. How-to post ideas assume you have the knowledge to execute them — check each idea is feasible before briefing.
- Post counts are distributed across the planning period but without specific dates (you choose the exact timing). Add a scheduling node (Buffer, Hootsuite API) to turn this into scheduled drafts.
- Trend research is from Tavily's indexed web content — it picks up broadly trending topics but not real-time social trends or viral moments.

---

## Ideas

- [ ] Google Sheets output: write each post idea as a row in a Sheet for collaborative editing
- [ ] Buffer/Hootsuite integration: push approved posts directly to a scheduling queue
- [ ] Monthly regeneration: schedule to run the first day of each month, auto-send to the content team
- [ ] Competitor gap analysis: search what competitors are posting and suggest differentiating angles

---

## License

MIT.
