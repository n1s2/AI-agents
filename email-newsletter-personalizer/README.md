# email-newsletter-personalizer

Sending the same newsletter to every subscriber treats a first-time free user the same as a five-year enterprise customer. The content is often relevant to both, but the framing, subject line, opening hook, and CTA emphasis that will resonate are completely different. This takes a single newsletter draft and generates a personalized version for each of your subscriber segments — same core content, different frame.

---

## What it does

Takes the newsletter content, up to 8 subscriber segment definitions (with descriptions, size, and sample interests), and brand voice. Claude produces per-segment versions, each with:
- Subject line calibrated to that segment
- Preview/preheader text
- Personalized opening hook (2–3 sentences)
- Content emphasis (which parts of the newsletter matter most to this segment)
- CTA framing (how to position the main call to action)
- Closing line before sign-off
- Personalization rationale

Also produces a default subject line for unclassified subscribers. Builds a side-by-side HTML comparison of all versions.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/personalize-newsletter \
  -H "Content-Type: application/json" \
  -d '{
    "newsletter_title": "The Build — Issue 34",
    "newsletter_goal": "Drive trial starts and share new integration announcement",
    "brand_voice": "Direct, practical, no fluff. We treat our readers as smart operators.",
    "newsletter_content": "This week: we shipped native Notion integration, our guide to running async standups is getting shared a lot, and we have a case study from Beacon Logistics on how they cut their planning overhead by 40%. Plus the usual roundup of tools and reads.\n\n[Notion integration] After months of requests, Flowdesk now syncs tasks bidirectionally with Notion. Works out of the box, no config needed.\n\n[Guide] How to run async standups that people actually read — we interviewed 12 ops leads about what works.\n\n[Case study] Beacon Logistics: from 3-hour weekly planning meetings to a 20-minute async check-in.\n\n[CTA] Start your free trial and connect Notion in under 5 minutes.",
    "reply_email": "marketing@flowdesk.com",
    "subscriber_segments": [
      {"segment_id": "new-free", "segment_name": "New free users (< 30 days)", "description": "Just signed up, evaluating the product", "size": 1840, "sample_interests": ["onboarding", "quick wins", "how-to guides"]},
      {"segment_id": "active-pro", "segment_name": "Active Pro subscribers", "description": "Paying, engaged weekly, power users", "size": 620, "sample_interests": ["new features", "integrations", "advanced workflows"]},
      {"segment_id": "dormant", "segment_name": "Dormant subscribers", "description": "Subscribed 3+ months, low engagement, never converted", "size": 2100, "sample_interests": ["unknown — need re-engagement"]},
      {"segment_id": "enterprise-leads", "segment_name": "Enterprise prospects", "description": "Company size 100+, in sales pipeline or downloaded enterprise guide", "size": 290, "sample_interests": ["case studies", "ROI", "team adoption", "security"]}
    ]
  }'
```

**Required:** `newsletter_content`, `subscriber_segments` (non-empty array)

---

## What changes per segment, what doesn't

The core newsletter content stays the same. What changes: subject line, preview text, opening hook, which content gets emphasized in the body, how the CTA is positioned, and the closing line. This is framing, not rewriting — the Notion integration announcement exists in all versions, but an enterprise prospect's version leads with "Beacon Logistics cut planning time 40%" while a new free user's version leads with "set up in 5 minutes."

---

## Segment limit

Up to 8 segments per call. For more segments, split into multiple calls.

---

## Limitations

- This generates copy components, not full rendered emails. You assemble the final emails in your ESP (Mailchimp, Klaviyo, etc.) using the per-segment subject lines, opening hooks, and CTA framing.
- Personalization quality depends on segment description quality. "Power users" produces better framing than "Segment B."

---

## License

MIT.
