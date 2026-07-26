# pricing-experiment-designer

Pricing changes are high-stakes and easy to get wrong. Random price increases without a hypothesis, testing too many things at once, insufficient sample sizes, and forgetting to monitor churn are all common failure modes. This designs structured pricing experiments with a clear hypothesis, control/variant setup, statistical rigor, guardrail metrics, and a phased rollout plan — including what NOT to test.

---

## What it does

Takes product name, current pricing, experiment goal, target segment, current metrics, customer feedback, and competitor pricing. Claude designs:

- **Experiment summary** and **hypothesis** ("If we [change], then [metric] will [direction] because [reason]")
- **Experiments** — each with: type (price_point/packaging/anchor/decoy/free_trial/freemium/annual_discount/per_seat_vs_flat), control vs variant, rationale, expected impact, risk level (low/medium/high), risk notes, implementation complexity, and implementation notes
- **Primary metric** — what to measure, how, baseline, minimum detectable effect
- **Secondary metrics** — other things to track
- **Sample size guidance** — visitors needed, estimated duration in weeks, confidence level, calculation notes
- **Guardrail metrics** — metrics that should NOT move (churn, NPS, support volume)
- **Rollout plan** — phase 1 (start small), phase 2 (expand), rollback criteria
- **Pricing psychology notes** — relevant principles for this experiment
- **What not to test** — experiments that seem logical but would backfire

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/design-pricing-experiment \
  -H "Content-Type: application/json" \
  -d '{
    "product_name": "Flowdesk",
    "experiment_goal": "increase_arpu",
    "currency": "USD",
    "monthly_visitors": 8000,
    "current_conversion_rate": 0.04,
    "current_pricing": "Single plan: $12/user/month. Free trial: 14 days, no credit card. No annual option. No free tier.",
    "current_metrics": "MRR: $52k. ARPU: $1,800/year. Avg team size: 12 users. Conversion free-to-paid: 4%. Churn: 3.2%/month.",
    "customer_feedback": "Multiple NPS responses mention price feels low for the value. 3 enterprise inquiries in last 90 days asking about annual contracts. 2 churned customers cited not enough features for the price tier they were on.",
    "competitor_pricing": "Asana: $10.99-$24.99/user/month. Monday.com: $9-$16/user/month. Both offer annual discounts of 15-18%.",
    "constraints": "Cannot add features to create tier differentiation in the next 60 days. Must stay under $20/user to remain competitive with Monday.com.",
    "reply_email": "growth@flowdesk.com"
  }'
```

**Required:** `product_name`, `current_pricing`

---

## Experiment goals

`increase_arpu`, `improve_conversion`, `reduce_churn`, `test_new_tier`, `optimize_packaging`, `willingness_to_pay`

Goal shapes what experiments Claude designs. Conversion improvement suggests testing free trial length, onboarding friction, or lower entry pricing. ARPU improvement suggests annual plans, tier introduction, or anchoring. Churn reduction suggests value-metric pricing or better plan-feature fit.

---

## Guardrail metrics

Every experiment includes metrics that should NOT move. A price increase that improves revenue but spikes churn by 2% is a failure. Claude always includes churn, NPS, and support volume as guardrails and flags what change in each metric should trigger rollback.

---

## What not to test

Claude explicitly lists experiments that seem logical for this product but would backfire. For example: "Don't test freemium — your product requires team adoption to show value, and single-user free accounts create noise without converting." This saves teams from chasing experiments that look good on paper but don't fit the product.

---

## Statistical rigor

Sample size guidance calculates visitors needed based on current conversion rate, expected effect size, and 95% confidence level. For low-traffic sites, Claude flags when a meaningful experiment would take too long to run and suggests workarounds (e.g. user research instead of A/B testing).

---

## Limitations

- Experiment designs are based on inputs you provide. Actual sample size calculations require your real conversion rate and baseline metrics — pass these for accurate estimates.
- This designs experiments; running them requires an A/B testing platform (Optimizely, LaunchDarkly, or custom implementation).

---

## License

MIT.
