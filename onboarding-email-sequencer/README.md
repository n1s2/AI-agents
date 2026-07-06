# onboarding-email-sequencer

Most onboarding email sequences are feature tours. "Did you know you can do X? Here's how to do Y!" — sent too frequently, too early, with no understanding of where users actually drop off. This designs a sequence around your specific activation actions, with one goal per email, suppression logic to skip emails when users already completed the action, and timing based on your sequence window.

---

## What it does

Takes product name, target persona, key activation actions (in priority order), sequence length, and email count. Optionally: known drop-off reasons and existing engagement data. Claude designs a complete sequence where each email has:

- Send day and trigger type (immediate/day N/action-based)
- Subject line and preview text
- Clear single goal
- Opening line (not generic — specific to the email's purpose)
- Body outline in bullet points
- Single CTA with destination
- Personalization tokens to include
- Skip-if condition (suppress when user already did the thing)

Also returns: sequence strategy rationale, A/B test suggestions, and success metrics.

HTML output shows each email as a card with day badge, goal, opening line, CTA, and skip logic.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-onboarding-sequence \
  -H "Content-Type: application/json" \
  -d '{
    "product_name": "Flowdesk",
    "product_description": "Lightweight project management for small teams.",
    "target_persona": "Operations manager at a 5-25 person company, switching from spreadsheets or an overcomplicated PM tool",
    "sequence_length_days": 21,
    "emails_in_sequence": 6,
    "tone": "helpful_and_direct",
    "activation_goal": "User has created their first project, added 3+ tasks, and invited at least one teammate",
    "key_actions": [
      "Create first project",
      "Add 3 tasks to the project",
      "Invite a teammate",
      "Connect Slack integration",
      "Complete first weekly check-in"
    ],
    "common_dropoff_reasons": "Users create an account but never create a project. Second common dropoff: created project but never invited anyone — using it solo which leads to churn within 60 days.",
    "existing_engagement_data": "Day 1 open rate 68%, day 3 drops to 34%, day 7 drops to 19%. CTA click rate averages 12%.",
    "reply_email": "growth@flowdesk.com"
  }'
```

**Required:** `product_name`, `target_persona`, `key_actions`, `sequence_length_days`

---

## One action per email

Claude is instructed that each email has one goal, one CTA. Multi-CTA emails dilute attention and reduce click rates. The sequence is designed around getting users through activation steps in order — not showcasing features.

---

## Skip-if logic

Every email includes a `skip_if` condition describing when the email should be suppressed. Your ESP needs to implement this via event-based suppression — but the logic is defined for you. A "create your first project" email should be suppressed if the user already created one.

---

## Limitations

- This designs the sequence content and timing. Actual sending and suppression logic must be implemented in your ESP (Klaviyo, Customer.io, Intercom, etc.).
- Sequence is for a single persona. For multiple personas (developer vs admin vs end user), run the agent separately for each.

---

## License

MIT.
