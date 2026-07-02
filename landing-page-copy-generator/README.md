# landing-page-copy-generator

Most landing page copy fails the same way: it describes the product instead of the outcome, has three competing CTAs, and never addresses the visitor's actual hesitation. This generates a complete landing page copy structure — hero, problem/solution framing, feature blocks written as benefits, objection handling, and closing CTA section — calibrated to your page type and audience.

---

## What it does

Takes product name, value proposition, target audience, page type, features, social proof, pricing, objections to address, and competitor angle. Claude writes:
- Hero: headline (outcome-focused, under 10 words), subheadline, CTA text
- Social proof bar (using only what you provided — never invented)
- Problem section (named from the visitor's perspective)
- Solution section
- Feature blocks (each headline is a benefit, not a feature label)
- Objection handling (objection + how the copy addresses it)
- Final CTA section
- SEO meta title and description
- Copy rationale explaining the strategic choices

Builds an HTML preview that resembles the actual page layout. Emails if requested.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-landing-copy \
  -H "Content-Type: application/json" \
  -d '{
    "product_name": "Flowdesk",
    "value_proposition": "Project management that takes 5 minutes to set up instead of 5 weeks. Built for teams who tried Asana and gave up.",
    "target_audience": "Operations managers at 5-25 person companies frustrated with overcomplicated PM tools",
    "page_type": "free_trial",
    "features": ["Drag-and-drop task boards", "Slack and email notifications", "No training required", "Free for up to 5 users"],
    "social_proof": "4,200+ teams, 4.8/5 on G2 from 1,100 reviews",
    "pricing_info": "Free up to 5 users, $8/user/month after",
    "main_cta": "Start free trial",
    "objections_to_address": ["Will my team actually switch tools again", "Is this too simple for our needs"],
    "tone": "confident_clear",
    "reply_email": "marketing@flowdesk.com"
  }'
```

**Required:** `product_name`, `value_proposition`, `target_audience`

---

## Page types

`saas_homepage`, `product_launch`, `lead_gen`, `event_registration`, `app_download`, `waitlist`, `free_trial`

---

## License

MIT.
