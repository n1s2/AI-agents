# brand-voice-analyzer

"Our voice is friendly, authentic, and professional" describes every brand and helps no writer. Real brand voice guidance comes from analyzing actual content — what words appear consistently, what sentence structures they favor, where they fall on formal-to-casual and serious-to-playful spectrums, and crucially, what they avoid. This analyzes up to 8 content samples and produces specific, usable voice guidance including a do/don't list with examples and a before/after rewrite demonstration.

---

## What it does

Takes brand name and content samples (from any source: website, emails, social, ads, docs). Claude analyzes and produces:

- **Voice summary** — what this brand actually sounds like based on the samples (honest, not the marketing version)
- **Voice attributes** — each with strength (strong/present/weak), a specific evidence quote from the samples, and practical writing implication
- **Tone spectrum** — 0–10 scales across: formal↔casual, serious↔playful, authoritative↔humble, distant↔warm
- **Vocabulary patterns** — words they use, words they avoid, sentence length, punctuation style
- **Do/Don't rules** — each with an example of what it looks like in practice
- **Inconsistencies** — where voice varies across samples, which samples are involved, and what to do about it
- **Persona descriptors** — if this brand were a person, what adjectives describe them
- **Differentiation** — how this voice differs from typical industry voice
- **Writing example** — same sentence written generically vs in brand voice (before/after)

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/analyze-brand-voice \
  -H "Content-Type: application/json" \
  -d '{
    "brand_name": "Flowdesk",
    "industry": "B2B SaaS / Project management",
    "target_audience": "Operations managers at small companies",
    "aspirational_voice": "We want to sound like a smart, direct colleague — not a vendor",
    "competitor_voice": "Asana sounds polished and corporate. Monday.com sounds energetic and sales-y. We want to be different from both.",
    "reply_email": "brand@flowdesk.com",
    "content_samples": [
      {"source": "homepage_hero", "content": "Flowdesk keeps your ops team moving. No training required, no IT ticket needed. Set up in an hour, not a quarter."},
      {"source": "email_subject_lines", "content": "Your team is losing 3 hours/week to this. Here is how we fixed it. One thing we wish we had built sooner. Most ops teams hit this wall around 25 people."},
      {"source": "blog_intro", "content": "If your ops playbook lives in a spreadsheet, you already know the problem. Someone updates the wrong version. The source of truth is wherever Sarah last saved it. You spend your Monday morning reconstructing what actually happened last week."},
      {"source": "onboarding_email", "content": "Welcome to Flowdesk. You have got 14 days to see if this works for your team. Here is what we recommend: spend the first 30 minutes setting up your first project. Not all your projects — just one. Pick the messiest one."},
      {"source": "error_message", "content": "Something went wrong on our end. We have been notified and will take a look. In the meantime, try refreshing the page. If the problem persists, reply to this message and a real human will help."}
    ]
  }'
```

**Required:** `brand_name`, `content_samples`

---

## Content sample format

Pass samples as an array of objects with `source` and `content`, or as plain strings. Sources help identify inconsistencies across channels (e.g., "homepage sounds different from onboarding emails").

```json
{"source": "homepage", "content": "..."}
// or just a string:
"Copy the text here directly"
```

Up to 8 samples, each up to 1,000 characters. More samples = more reliable analysis.

---

## Inconsistency detection

Claude identifies where voice varies across samples — common patterns: formal on the website, casual on social; confident in marketing copy, hedging in onboarding; consistent personality that breaks down in error messages. Each inconsistency includes which samples are involved and a recommendation.

---

## Before/after example

The `writing_examples` section shows the same information written generically vs in brand voice. This is the most immediately usable output — writers can reference it as a concrete demonstration of what the voice analysis means in practice.

---

## Limitations

- Analysis is based on samples provided — 2 samples produces a thin analysis; 6–8 produces reliable patterns.
- Voice analysis reflects what's written, not what the brand intends. If there's a gap between aspiration and reality, pass the `aspirational_voice` field and Claude will note the gap.

---

## License

MIT.
