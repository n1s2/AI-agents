# social-media-reply-generator

Replying to social media at scale — especially when posts include complaints, questions, and mentions mixed together — is slow when done manually and inconsistent when delegated to junior staff. This generates platform-appropriate replies for a batch of posts in one call, flags escalations and crisis posts for human review, and identifies high-value accounts worth prioritizing.

---

## What it does

Takes up to 20 social media posts with platform, author, content, and category. Claude writes a reply for each — platform-appropriate length and tone (Twitter is punchy, LinkedIn is professional, Instagram is warm) that references what the specific person said, not a generic canned response. Each reply also gets:

- **Reply strategy** — what approach was taken and why
- **Escalate flag** — set true if this post needs human eyes before posting
- **Do not post flag** — set true if Claude can't safely reply (legal risk, sensitive topic, insufficient info)
- **Character count**

Also returns: crisis posts list (post IDs needing immediate human review), high-value engagement list (verified/high-follower accounts to prioritize), and a batch sentiment summary.

For posts flagged for escalation: fires an immediate plain-text alert to the escalation email.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-social-replies \
  -H "Content-Type: application/json" \
  -d '{
    "brand_name": "Flowdesk",
    "brand_description": "Lightweight project management for small teams",
    "brand_voice": "Direct and friendly — we are a small team that genuinely cares about our users",
    "do_not_mention": ["pricing", "competitors by name", "upcoming features not yet announced"],
    "escalation_email": "social@flowdesk.com",
    "reply_email": "social@flowdesk.com",
    "posts": [
      {"post_id": "TW-001", "platform": "twitter", "author_handle": "sarahops", "post_content": "Love how fast @flowdesk is compared to Asana. Set up our whole team in like 20 minutes", "category": "positive", "follower_count": 1200},
      {"post_id": "TW-002", "platform": "twitter", "author_handle": "frustrated_pm", "post_content": "@flowdesk your mobile app keeps crashing when I try to open attachments. Happening for 3 days now", "category": "complaint", "urgent": true},
      {"post_id": "LI-001", "platform": "linkedin", "author_handle": "jordan_ops", "author_name": "Jordan Adesanya", "post_content": "Has anyone used Flowdesk for a team of 50+ people? Would love to hear from ops leaders who have scaled it", "category": "question", "follower_count": 8400, "verified": true},
      {"post_id": "TW-003", "platform": "twitter", "author_handle": "techcrunch_reporter", "post_content": "Hearing rumors @flowdesk is being acquired. Any comment?", "category": "crisis", "follower_count": 45000, "verified": true, "urgent": true}
    ]
  }'
```

**Required:** `posts`, `brand_name`

---

## Post categories

`positive`, `negative`, `question`, `complaint`, `feature_request`, `mention`, `crisis`, `neutral`

Category affects reply tone and approach. Complaints get acknowledgment and a path to resolution. Questions get helpful answers. Crisis posts get escalated automatically and get a careful holding response.

---

## Escalation and do-not-post

**Escalate** — draft reply is included but should be reviewed by a human before posting. Fires an email alert with the post content, escalation reason, and draft reply.

**Do not post** — Claude cannot safely generate a reply for this post (acquisition rumor, pending legal matter, insufficient brand information). The reply card shows the reason and the reply text is struck through.

---

## Crisis handling

Posts categorized as `crisis` are automatically added to the `crisis_posts` list. Claude generates a careful holding response ("we're looking into this") rather than an engagement reply. The crisis_posts list in the response tells your team which posts need immediate human attention.

---

## Limitations

- Replies are drafts — review before posting, especially for complaints and crisis posts. The model can misread tone or context.
- This generates replies for the posts you submit; it doesn't monitor social platforms directly. Connect to a social listening tool (Mention, Sprout Social) to pull posts into this agent automatically.

---

## License

MIT.
