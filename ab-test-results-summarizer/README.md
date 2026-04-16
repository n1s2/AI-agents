# FLOOWBOX - A/B Test Results Summarizer

Most people look at A/B test results and just compare percentages. This workflow runs proper statistical significance testing, calculates confidence levels, and gives a clear verdict in plain English — along with what to test next.

## What it does

Send A/B test data via webhook — visitor and conversion counts for both variants. The code node runs a two-proportion Z-test to calculate statistical significance, confidence level, lift percentage, and winner. GPT-4o interprets the statistical results in business terms, provides a clear verdict (implement B / stick with A / run longer / inconclusive), explains what was learned, and suggests next test ideas. Saves to a Notion test log and posts to Slack.

## Tools Used
- **Orchestration:** n8n
- **Statistical Testing:** Code node (Z-test for proportions)
- **Interpretation:** OpenAI GPT-4o
- **Test Log:** Notion
- **Report:** Slack
- **Trigger:** Webhook

## Statistical calculation

The workflow implements a two-proportion Z-test:

```
p_pool = (conversions_A + conversions_B) / (visitors_A + visitors_B)
SE = sqrt(p_pool * (1 - p_pool) * (1/n_A + 1/n_B))
Z = |p_A - p_B| / SE

Confidence: Z > 2.576 → 99%, Z > 1.96 → 95%, Z > 1.645 → 90%
```

## Verdict options

| Verdict | Meaning |
|---|---|
| `implement_B` | B wins at ≥95% confidence — ship it |
| `stick_with_A` | A wins — keep current version |
| `run_longer` | Directional result but not yet significant |
| `inconclusive` | No clear signal — rethink the test |

## Example output

```json
{
  "verdict": "implement_B",
  "confidence_interpretation": "95% confident this is a real effect, not random variation",
  "business_impact": "At current traffic, implementing B adds ~28 conversions/week",
  "what_we_learned": ["Shorter form reduces friction significantly", "Users respond to social proof near CTA"],
  "next_test_ideas": ["Test button color on the winning variant", "A/B test the headline above the form"]
}
```

## Webhook payload

```json
{
  "test_name": "Checkout Form Length Test",
  "hypothesis": "Reducing form from 8 fields to 4 will increase checkout completion",
  "variant_a": {"visitors": 2400, "conversions": 156, "description": "8-field form (control)"},
  "variant_b": {"visitors": 2380, "conversions": 198, "description": "4-field form (treatment)"},
  "duration_days": 14,
  "primary_metric": "checkout_completion"
}
```

## Why I built this

Clients were making implementation decisions based on raw percentage differences with no regard for sample size or statistical significance. A test with 50 visitors showing "B is 20% better" means nothing — this workflow tells them exactly when to trust the results.

## Setup

1. OpenAI API key
2. Notion integration + Test Log DB
3. Slack Bot Token + #analytics channel
