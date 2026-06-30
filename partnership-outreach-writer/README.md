# partnership-outreach-writer

Partnership outreach fails in predictable ways: "I think there's synergy between our companies" tells the recipient nothing. Leading with what you want instead of what they get. No specificity about what the partnership actually means in practice. Generic subject lines that get ignored.

This writes partnership outreach calibrated to the partnership type and both companies' positions. Searches for recent context about the partner company via Tavily for personalization, then Claude writes a specific email, a different-angled LinkedIn message, a 3–4 bullet partnership pitch for when they respond, and a follow-up angle for non-responders.

---

## What it does

1. Accepts: your company, partner company, partnership type, your name/title, value prop, mutual benefit, partner context, existing relationship
2. Searches Tavily for recent news about the partner company's partnership programs
3. Claude writes:
   - Personalization hook (specific thing about the partner from research)
   - Email: subject line + body (under 200 words, leads with partner benefit, specific ask)
   - LinkedIn message (different angle from email)
   - Partnership pitch (3–4 bullets for when they respond wanting detail)
   - Follow-up angle (different approach if first message is ignored)
   - Quality note (honest assessment of how strong this opportunity looks)
4. Builds HTML output with all assets
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
curl -X POST https://your-n8n.com/webhook/write-partnership-outreach \
  -H "Content-Type: application/json" \
  -d '{
    "your_company": "Flowdesk",
    "your_description": "Lightweight project management for small teams. 4,200 customers, integrates with Slack, GitHub, and Google Workspace.",
    "your_name": "Maya Chen",
    "your_title": "Head of Partnerships",
    "partner_company": "Notion",
    "partner_contact": "Alex Rivera",
    "partner_title": "Head of Partnerships",
    "partner_description": "All-in-one workspace for notes, docs, and project management",
    "partnership_type": "technology_integration",
    "channel": "email",
    "value_proposition": "A native Flowdesk integration in the Notion integration gallery would let Notion users manage their project tasks in Flowdesk while keeping documentation in Notion. Our overlap audience is SMBs who use both tools today with manual workarounds.",
    "mutual_benefit": "Flowdesk gets distribution to Notion's user base. Notion fills a gap in their project management integration offering for teams who want something simpler than Jira.",
    "partner_context": "Notion recently expanded their integration partner program.",
    "tone": "professional_warm",
    "reply_email": "maya@flowdesk.com"
  }'
```

**Required:** `your_company`, `partner_company`, `partnership_type`, `your_name`, `value_proposition`

---

## Partnership types

`co_marketing`, `technology_integration`, `reseller`, `referral`, `distribution`, `content`, `strategic_alliance`, `agency_partner`, `affiliate`

Each type gets different framing. A technology integration pitch leads with the user experience benefit. A co-marketing pitch leads with audience overlap. A reseller pitch leads with revenue potential for the partner.

---

## Channel options

`email`, `linkedin`, `both` — both drafts the email and LinkedIn message so you can choose which channel to use first.

---

## Leading with partner benefit

The most important principle: what does the partner company get? Not what you want — what they get. Claude is explicitly instructed to lead with the partner benefit, not your company's story.

---

## Limitations

- Tavily research may miss private partnership programs or very recent announcements. Supplement `partner_context` with anything you already know.
- Verify the personalization hook is accurate before sending — wrong personalization is worse than none.

---

## License

MIT.
