# knowledge-base-article-writer

Writing KB articles is one of the most time-consuming support tasks. A good article requires understanding what the user is actually trying to do, structuring steps clearly, anticipating mistakes, and keeping it scannable. Most are too long, too feature-focused, or don't address the errors users actually hit. This generates a complete, structured KB article from expert notes and feature descriptions.

---

## What it does

Takes article topic, product name, article type, target audience, expert notes, common errors, feature description, and related articles. Claude writes:

- Action-oriented title and SEO meta description
- Estimated read time
- Intro (what the user will be able to do after reading)
- Prerequisites (what they need before starting)
- Sections with clear headings, numbered steps where procedural, screenshot placeholders, and callout boxes
- Common mistakes section (what goes wrong + how to fix it)
- Next steps
- Related articles
- Internal notes for the reviewer (accuracy checks, screenshots needed, links to add)
- Suggested KB tags

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/write-kb-article \
  -H "Content-Type: application/json" \
  -d '{
    "product_name": "Flowdesk",
    "article_topic": "How to invite team members and set their permissions",
    "article_type": "how_to",
    "target_audience": "end_users",
    "search_keywords": ["invite team", "add user", "team permissions"],
    "prerequisite_knowledge": "You must be a Workspace Admin to invite team members",
    "feature_description": "Workspace Admins can invite users by email from Settings > Team Members > Invite. Roles: Admin, Member, Viewer.",
    "common_errors": "Most common: invite email goes to spam. Second: user signs up from homepage instead of invite link, creating a separate account.",
    "reply_email": "docs@flowdesk.com"
  }'
```

**Required:** `article_topic`, `product_name`

---

## Article types

`how_to`, `troubleshooting`, `concept_explainer`, `reference`, `faq`, `release_notes_summary`, `best_practices`

---

## Limitations

The article is a first draft — always review for product accuracy before publishing. Pass `expert_notes` or `feature_description` for best results; without them Claude writes from the topic alone and may miss product-specific details.

---

## License

MIT.
