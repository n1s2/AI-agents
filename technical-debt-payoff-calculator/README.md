# technical-debt-payoff-calculator

"This tech debt is really slowing us down" doesn't win a prioritization fight against a feature with a projected revenue number attached. This calculates the actual annual cost of a debt item deterministically (engineer time lost, incident cost, blocked revenue) versus the cost to fix it, computes payback period and 3-year net benefit, then has Claude translate the math into a business case — including being honest when the numbers don't actually support prioritizing the fix.

---

## What it does

Takes the debt item description, current costs (engineer hours lost per week, incidents per quarter, average incident cost in hours, blocked revenue estimate), hourly engineer cost, and payoff effort in days. The validator computes deterministically:

- Weekly and annual cost from lost engineer time
- Annual cost from incidents
- Total annual cost of not fixing (including blocked revenue)
- Payoff cost (effort × hourly rate)
- Payback period in weeks
- 3-year net benefit

Claude then explains the result:

- **Business case summary** — plain language for a non-engineer executive, honest even when weak
- **Verdict** — strong_case/moderate_case/weak_case/insufficient_data with rationale
- **Payback interpretation** — what the payback period means practically
- **Risks not captured in the math** — qualitative factors like team morale, key-person risk, compounding technical risk
- **Caveats** — assumptions leadership should know about
- **Comparison framing** — how to position this against competing priorities
- **Recommended narrative** — a 2-sentence pitch for a prioritization meeting

HTML report with cost/payoff/payback stat cards, 3-year net benefit highlighted, and qualitative risk callouts.

---

## Stack

n8n (Code node for deterministic financial math), Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/calculate-debt-payoff \
  -H "Content-Type: application/json" \
  -d '{
    "debt_item": "Migrate task assignment service off the monolith to a dedicated service",
    "currency": "USD",
    "hourly_engineer_cost": 95,
    "payoff_effort_days": 15,
    "team_context": "4-person backend team, this service is on the critical path for the bulk operations feature launching next quarter",
    "reply_email": "priya@flowdesk.com",
    "current_costs": {
      "engineer_hours_per_week_lost": 6,
      "incidents_per_quarter": 2,
      "avg_incident_cost_hours": 5,
      "blocked_revenue_estimate": 0
    }
  }'
```

**Required:** `debt_item`, `current_costs`

---

## Deterministic financial math

All dollar calculations happen in a Code node with real arithmetic:

```
weekly_lost_cost = engineer_hours_per_week_lost × hourly_engineer_cost
annual_lost_cost_from_time = weekly_lost_cost × 52
annual_incident_cost = incidents_per_quarter × 4 × avg_incident_cost_hours × hourly_engineer_cost
total_annual_cost = annual_lost_cost_from_time + annual_incident_cost + blocked_revenue_estimate
payoff_cost = payoff_effort_days × 8 × hourly_engineer_cost
payback_weeks = (payoff_cost / total_annual_cost) × 52
three_year_net_benefit = (total_annual_cost × 3) - payoff_cost
```

Claude explains and contextualizes these numbers — it does not compute them, keeping the actual math auditable and trustworthy.

---

## Honest about weak cases

If the payback period is long or the annual cost is small relative to the fix cost, Claude is instructed to say so plainly rather than spin a weak case into a strong one. The `weak_case` verdict exists precisely so this tool doesn't become a rubber stamp for every debt item someone wants to prioritize.

---

## What the math misses

Some of the most important debt-payoff factors — team morale, key-person risk, the way debt compounds and makes future changes progressively harder — don't reduce cleanly to a dollar figure. The `risks_not_captured_in_math` field exists to surface these explicitly rather than let the dollar number be treated as the whole story.

---

## Limitations

- Estimates are only as good as the inputs. "Engineer hours lost per week" is inherently an estimate — get team consensus on a reasonable number rather than a single person's guess.
- This is a decision-support tool, not a mandate. Use it to structure the conversation, not to remove human judgment from prioritization.

---

## License

MIT.
