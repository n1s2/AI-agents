# email-sequence-optimizer

Email sequences that underperform usually have the same problems: subject lines that beg for the open instead of earning it, emails that ask before delivering value, pacing that doesn't match where the reader is in their journey, and no coherent arc across the sequence. This takes your existing sequence and returns complete rewrites — not suggestions, but finished emails ready to test — plus A/B test ideas, timing recommendations, and missing email gaps.

---

## What it does

Takes up to 15 emails (each with subject, preview text, body, send delay, and optionally open/click/unsubscribe/conversion rates). Claude produces:

- **Sequence assessment** — what it does well, where it loses people, biggest opportunity
- **Sequence-level issues** — pacing problems, goal conflicts, missing emails
- **Optimized emails** — each with: rewritten subject, preview text, and complete email body, plus specific changes made and why, and performance prediction
- **Structure recommendations** — high/medium/low priority structural changes
- **A/B test suggestions** — specific tests per email (subject/opening line/CTA/send time) with control/variant/hypothesis
- **Send timing recommendations** — delays between emails with rationale
- **Missing emails** — gaps in the sequence with suggested purpose and trigger

HTML report shows each email as a card with original subject vs optimized, full rewritten body in a rendered block, and change notes.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/optimize-email-sequence \
  -H "Content-Type: application/json" \
  -d '{
    "sequence_name": "Trial Conversion Sequence",
    "sequence_type": "trial_conversion",
    "product_name": "Flowdesk",
    "audience_description": "New trial users who signed up but have not created their first project in 48 hours",
    "sequence_goal": "Get users to create their first project and invite a team member within 14 days",
    "brand_voice": "Direct and practical — we trust users to be smart",
    "current_performance_notes": "Email 1 open rate is 52% but click rate is only 4%. Emails 3 and 4 have very low open rates suggesting the sequence is losing people early.",
    "reply_email": "growth@flowdesk.com",
    "emails": [
      {
        "position": 1,
        "subject": "Welcome to Flowdesk!",
        "preview_text": "Get started in minutes",
        "send_delay": "immediately on signup",
        "goal": "Get first login and project creation",
        "open_rate": 52,
        "click_rate": 4,
        "body": "Hi there,\n\nWelcome to Flowdesk! We are so excited to have you here.\n\nFlowdesk is the best project management tool for small teams. You can create projects, assign tasks, and track progress all in one place.\n\nTo get started, just click the button below to create your first project.\n\n[Create your first project]\n\nIf you have any questions, just reply to this email.\n\nThanks,\nThe Flowdesk Team"
      },
      {
        "position": 2,
        "subject": "Quick tip: how to set up your workspace",
        "send_delay": "day 2",
        "goal": "Drive feature adoption",
        "open_rate": 34,
        "click_rate": 6,
        "body": "Hi,\n\nHere is a quick tip to help you get the most out of Flowdesk.\n\nThe best way to start is to create a project for your most important current work. Then add your team members so everyone can see what is happening.\n\nMost teams get value in the first week when they replace their spreadsheet with Flowdesk.\n\n[Watch the 2-minute setup video]\n\nLet us know if you need help!\n\nThe Flowdesk Team"
      },
      {
        "position": 3,
        "subject": "Are you getting value from Flowdesk?",
        "send_delay": "day 5",
        "open_rate": 18,
        "click_rate": 2,
        "body": "Hi,\n\nJust checking in to see if you are getting value from Flowdesk.\n\nIf you have not had a chance to explore the platform yet, we would love to help. Book a quick call with our team and we will walk you through the best features for your use case.\n\n[Book a 15-minute call]\n\nThanks,\nThe Flowdesk Team"
      }
    ]
  }'
```

**Required:** `sequence_name`, `emails`

---

## Sequence types

`onboarding`, `nurture`, `sales_outreach`, `re_engagement`, `post_purchase`, `trial_conversion`, `event_followup`, `churn_prevention`, `upsell`

Type shapes the optimization. Trial conversion sequences should build momentum toward a specific activation moment. Re-engagement sequences need to acknowledge the gap without guilt-tripping. Churn prevention sequences should lead with value recall, not discounts.

---

## Performance data

Pass open rate, click rate, unsubscribe rate, and conversion rate per email if you have them. Claude uses this to identify where the sequence is losing people and diagnoses why. Without performance data, Claude optimizes based on best practices for the sequence type and audience.

---

## Complete rewrites

The `optimized_body` field is a complete, finished email — not a summary of changes or a template with placeholders. Copy it directly into your email platform. Subject lines and preview text are also fully rewritten.

---

## Limitations

- Rewrites are based on the body text you provide. If the original emails reference specific product features, pricing, or URLs that need to be preserved, review the optimized versions before deploying.
- For sequences with more than 15 emails, run the agent on subsets (e.g., emails 1–7, then 8–15) and synthesize the structural recommendations.

---

## License

MIT.
