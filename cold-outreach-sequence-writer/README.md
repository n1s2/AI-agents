# cold-outreach-sequence-writer

Most cold outreach fails for the same reasons: it's too long, it's about the sender not the recipient, every follow-up is a variation of "just checking in," and the subject lines are indistinguishable from 50 other emails in the inbox.

This generates multi-step outreach sequences — email and LinkedIn — that are actually structured to get replies. You describe who you are, what you do, who you're targeting, and what pain you solve. Claude writes each touchpoint as a distinct message with its own angle, not copies of the same pitch at different intervals.

The sequence includes objection handling scripts, a list of what to research before sending each prospect, and explicit notes on what not to do for this specific sender/target combination.

---

## What it does

1. Accepts a POST: sender details, value prop, target role and industry, known pain points, optional specific prospect/company, tone, sequence length, channel order
2. Claude writes a complete multi-step sequence with:
   - Each message as a standalone piece (email or LinkedIn)
   - Subject lines, send timing, and purpose per step
   - Writing notes explaining why each message is structured the way it is
   - Personalization hooks — what to research per prospect and where
   - Objection responses for the 3 most common pushbacks
   - What not to do for this specific combination
3. Formats into a clean HTML document
4. Emails the sequence if `reply_email` is provided
5. Returns full JSON in the webhook response

---

## Stack

- **n8n** — webhook + workflow
- **Anthropic Claude** (claude-opus-4-5) — sequence writing
- **SMTP** — optional email delivery

---

## Setup

### 1. Environment variables

```
FROM_EMAIL=sequences@yourdomain.com
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** (optional)

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/write-outreach \
  -H "Content-Type: application/json" \
  -d '{
    "sender_name": "Maya Chen",
    "sender_company": "Patchwork",
    "sender_role": "Head of Sales",
    "sender_value_prop": "Patchwork reduces engineering onboarding time by 60% by auto-generating internal documentation from your existing codebase. Teams using it cut time-to-first-PR for new hires from 3 weeks to 5 days.",
    "target_role": "VP of Engineering",
    "target_industry": "Series B SaaS",
    "target_pain_points": "Scaling eng teams fast, onboarding bottlenecks, documentation debt that nobody maintains",
    "specific_prospect": "Jordan Park",
    "specific_company": "Fleetio",
    "tone": "direct",
    "sequence_length": 4,
    "channel_order": ["email", "linkedin", "email", "email"],
    "social_proof": "Used by eng teams at Vercel, Linear, and Retool. Average team saves 8 engineer-hours per new hire in the first month.",
    "call_to_action": "20-minute demo",
    "reply_email": "maya@patchwork.io"
  }'
```

**Required:** `sender_name`, `sender_company`, `sender_value_prop`, `target_role`, `target_industry`

---

## Tone options

| Tone | Style |
|---|---|
| `direct` | Confident, no fluff, gets to the point in sentence one |
| `warm` | Friendly and human, still professional — reads like a colleague |
| `formal` | Conservative register, good for regulated industries or senior buyers |
| `casual` | Like two professionals texting — short, punchy |
| `challenger` | Opens with a provocative insight or contrarian observation to earn attention |

---

## Channel order

Pass channels as an array matching your sequence length:

```json
"channel_order": ["email", "linkedin", "email", "email"]
```

- `email` — full message with subject line, under 100 words
- `linkedin` — connection note or message, under 300 characters

The sequence is designed around the channel — LinkedIn messages are far shorter and more conversational than email.

---

## Generic vs personalized sequences

If you don't pass `specific_prospect` and `specific_company`, Claude writes for the archetype — a generic VP of Engineering at a Series B SaaS, for example. This gives you a template sequence you can adapt.

If you pass a specific name and company, Claude writes the sequence as if it's actually going to that person. The first message especially becomes much sharper when there's a real context to write toward.

---

## The personalization hooks section

This is often the most overlooked part of a sequence. Claude lists 3-5 specific things to research per prospect before sending — not "look them up on LinkedIn" but "check if they've posted in the last 30 days about hiring or team scaling," or "see if their company recently announced a funding round that would imply rapid headcount growth."

Good personalization isn't mentioning their company name in paragraph one. It's referencing something real.

---

## Writing notes

Each touchpoint includes a `writing_notes` field explaining why the message is structured the way it is — why this angle, why this length, what the message is trying to accomplish beyond the surface CTA. Useful if you're adapting the sequence or training a team.

---

## Follow-up philosophy

The sequence is built so each follow-up adds something new rather than restating the original message. Step 2 doesn't say "just following up." It opens a different door — a different angle on the value prop, a relevant insight, a case study reference, a direct question. If someone didn't respond to the first message, sending a slightly different version of the same message isn't a strategy.

---

## Limitations

- Claude doesn't know your specific prospects — it writes toward the archetype or the named person based on what you provide. Actual personalization (referencing a specific recent LinkedIn post, a news article about their company) still requires human research.
- Sequence quality improves significantly with a specific, differentiated value prop. "We help companies grow faster" produces worse sequences than "We reduce customer churn in mid-market SaaS by identifying at-risk accounts 30 days earlier."
- The webhook generates one sequence per call. For bulk sequence generation across a prospect list, call it in a loop from a parent workflow or build a Google Sheets integration on top.

---

## Ideas

- [ ] A/B variant mode: generate two versions of the first email with different angles, pick the better one
- [ ] Reply classifier: pair with an email webhook that categorizes inbound replies (interested / not now / unsubscribe / referral)
- [ ] Sequence library: save generated sequences to a sheet, tag by target persona, reuse and refine over time
- [ ] Performance tracking: log which sequences were used and annotate with reply rates after a campaign

---

## License

MIT.
