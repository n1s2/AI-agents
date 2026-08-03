# press-release-writer

Press releases that bury the news under three paragraphs of company background never get read past the headline by a busy reporter. AP-style press releases lead with the actual news, use the inverted pyramid (most important fact first), and have quotes that sound like a real person said them — not marketing copy with quotation marks around it. This generates a complete, properly structured press release from your key facts, and never invents facts or numbers beyond what you provide.

---

## What it does

Takes company name, announcement type, key facts, quote sources, and boilerplate. Claude writes:

- **Headline** — newsworthy and specific, not generic
- **Subheadline** — supporting context (optional)
- **Dateline and lead paragraph** — the single most important fact in who/what/when/where/why format
- **Body paragraphs** — supporting details in descending order of importance (inverted pyramid)
- **Quotes** — attributed to the people/roles you specify, written to sound natural rather than corporate
- **Boilerplate** — standard "About [Company]" paragraph
- **Media contact block**

Also generates: 2–3 alternative headlines, suggested distribution (which outlet types/reporters this suits based on announcement type), and SEO keywords for online posting.

HTML formatted as a classic press release document with serif type, "FOR IMMEDIATE RELEASE" header, and pull-quote styling.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/write-press-release \
  -H "Content-Type: application/json" \
  -d '{
    "company_name": "Flowdesk",
    "announcement_type": "funding",
    "dateline": "San Francisco, CA",
    "release_date": "2025-06-12",
    "key_facts": "Flowdesk raised a $12 million Series A led by Northbeam Ventures, with participation from Foundry Group and existing seed investors. The round brings total funding to $15.5 million. Flowdesk will use the funding to expand its engineering team from 9 to 20 people and accelerate its integrations roadmap, including partnerships with Notion, Slack, and Linear announced earlier this year. The company has grown from $200K to $1.2M in annual recurring revenue over the past 12 months, serving over 400 operations teams.",
    "key_metrics": "$1.2M ARR, 400+ customers, 6x revenue growth in 12 months",
    "industry_context": "Part of a broader trend of investment in vertical-specific operations tools as companies move away from general-purpose project management software",
    "quotes_from": [
      {"name": "Jake Reyes", "title": "CEO and Co-Founder, Flowdesk", "talking_points": "excited about the milestone, focused on serving ops teams specifically rather than being a generic PM tool, grateful to customers"},
      {"name": "Maria Chen", "title": "Partner, Northbeam Ventures", "talking_points": "impressed by the growth and retention numbers, believes in the vertical-specific approach, excited about the team"}
    ],
    "company_boilerplate": "Flowdesk is project management built for operations teams. Unlike general-purpose tools, Flowdesk is designed specifically for the workflows of ops-heavy businesses in logistics, professional services, and agencies. Founded in 2023, Flowdesk serves over 400 customers.",
    "media_contact": {
      "name": "Sam Torres",
      "title": "Head of Communications",
      "email": "press@flowdesk.com",
      "phone": "+1-555-0142"
    },
    "target_publications": ["TechCrunch", "VentureBeat", "SaaStr"],
    "reply_email": "press@flowdesk.com"
  }'
```

**Required:** `company_name`, `announcement_type`, `key_facts`

---

## Announcement types

`funding`, `product_launch`, `partnership`, `acquisition`, `leadership_hire`, `milestone`, `award`, `expansion`, `research_findings`

Type shapes the structure and suggested distribution. Funding announcements lead with the round size and investors. Product launches lead with the capability and customer benefit. Leadership hires lead with the person's background and why it matters now.

---

## Never fabricates facts

Claude is explicitly instructed to use only the facts provided — it does not invent additional statistics, dates, or claims. If you don't provide a specific number, the release won't include a made-up one. This is critical for press releases, where factual accuracy has legal and reputational consequences.

---

## Natural-sounding quotes

Quotes are written to sound like something the named person would actually say — not "We are thrilled to announce this exciting milestone" corporate boilerplate. Pass `talking_points` per quote source to guide what each person's quote should emphasize.

---

## Limitations

- This is a draft — review carefully before distribution, especially quotes (confirm with the actual person before attributing words to them) and any numbers or claims.
- Distribution and SEO suggestions are general guidance, not a substitute for a PR agency's media relationships and pitch strategy.

---

## License

MIT.
