# pricing-page-ab-tester

Most SaaS pricing page tests are guesses. "Let's try annual billing upfront" or "let's add a fourth plan" without a clear hypothesis about why it would work. This takes your current pricing structure, conversion context, and known problems, and produces prioritized A/B test hypotheses — each with a specific hypothesis, control vs variant, metric to measure, expected lift rationale, rough sample size needed, and implementation notes.

---

## What it does

Takes current pricing plans (names, prices, features, which is highlighted), target audience, conversion rate if known, conversion goal, known problems, and competitor pricing context. Claude produces:

- Pricing diagnosis (what the current structure gets right and wrong)
- Prioritized A/B tests — each with hypothesis, control, variant, metric, expected lift, sample size estimate, risk level, and implementation notes
- Pricing psychology observations (anchoring, decoy effects, feature grouping issues)
- Quick wins (changes that don't need a test — just do them)
- Copy suggestions: plan names, CTA button text, value anchoring approach
- What not to test (common tests that probably won't help given this specific context)

HTML report with control/variant side-by-side for each test.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/advise-pricing-ab \
  -H "Content-Type: application/json" \
  -d '{
    "product_name": "Flowdesk",
    "product_description": "Lightweight project management for small teams",
    "target_audience": "Ops managers at 5-25 person companies",
    "current_conversion_rate": 2.4,
    "avg_revenue_per_user": 18,
    "main_conversion_goal": "free_to_paid",
    "pricing_problems": "Most visitors leave without upgrading. Free plan may be too generous. Users say Pro seems expensive for their small team but we think the price is fair.",
    "competitor_pricing": "Asana Free/Premium at $13.49/user. Trello Free/Standard at $5/user. Monday from $9/user.",
    "reply_email": "growth@flowdesk.com",
    "current_pricing": [
      {"name": "Free", "price": 0, "currency": "USD", "interval": "month", "features": ["Up to 5 users", "Unlimited tasks", "Basic integrations"]},
      {"name": "Pro", "price": 12, "currency": "USD", "interval": "month", "features": ["Unlimited users", "Advanced reporting", "Priority support", "All integrations"], "highlighted": true},
      {"name": "Enterprise", "price": 0, "currency": "USD", "interval": "month", "features": ["Everything in Pro", "SSO", "Custom contracts", "Dedicated CSM"]}
    ]
  }'
```

**Required:** `product_name`, `current_pricing` (non-empty array)

---

## Test structure

Each test comes with:
- **Hypothesis** — the full "if we change X, we expect Y because Z" statement
- **Control vs variant** — exactly what changes
- **Metric** — the primary thing to measure (not "conversion" but "free-to-paid upgrade rate within 30 days")
- **Expected lift** — and why (not just a number)
- **Sample size** — rough estimate based on current conversion rate if provided
- **Risk level** — low (copy/positioning change), medium (pricing change), high (structural change)
- **Implementation notes** — what to actually build

---

## Quick wins vs tests

Not everything needs a test. Some changes have such clear directional benefit (removing friction from the upgrade flow, fixing misleading plan names, adding a monthly/annual toggle) that running a test is slower than just doing them. Claude separates these from genuine tests.

---

## License

MIT.
