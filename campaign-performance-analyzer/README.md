# campaign-performance-analyzer

A spreadsheet of CPA by channel tells you which number is bigger, not what to do about it. This analyzes campaign performance across channels, diagnoses why underperforming channels are underperforming (creative fatigue vs audience mismatch vs landing page vs bid strategy), identifies specific budget reallocation moves with numbers, and gives a prioritized action list.

---

## What it does

Takes campaign name, goal, target CPA/ROAS, and channel performance data (spend, impressions, clicks, CTR, conversions, conversion value, CPA, ROAS). Claude produces:

- **Performance summary** — overall performance, standout channel, biggest concern
- **Channel analysis** — each channel with: performance verdict (strong/adequate/underperforming/failing), actual CPA/ROAS, comparison to target, key insight, and recommendation (scale/maintain/optimize/pause/test_creative)
- **Budget reallocation** — specific from/to channel moves with amount, rationale, and expected impact
- **Underperformance diagnosis** — for each weak channel: likely cause (creative_fatigue/audience_mismatch/landing_page/bid_strategy/competition/seasonality), evidence, and suggested test
- **Scaling opportunities** — channels worth increasing spend on, with rationale and risk
- **Blended metrics** — overall CPA/ROAS across all channels vs target
- **Next actions** — prioritized as immediate/this_week/next_cycle with expected outcome

HTML report with channel cards, reallocation arrows (from → to), and diagnosis cards.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/analyze-campaign-performance \
  -H "Content-Type: application/json" \
  -d '{
    "campaign_name": "Q2 Trial Signup Campaign",
    "campaign_goal": "Drive trial signups for Flowdesk at target CPA under $85",
    "campaign_period": "April 1 - June 30, 2025",
    "currency": "USD",
    "target_cpa": 85,
    "target_roas": 3.5,
    "audience_context": "Operations managers at 15-100 person companies, US market",
    "reply_email": "growth@flowdesk.com",
    "channels": [
      {"channel": "Google Search", "spend": 18500, "impressions": 420000, "clicks": 8900, "ctr": 2.1, "conversions": 215, "conversion_value": 92450, "cpa": 86.05, "roas": 5.0, "notes": "Consistent performer, mostly branded + high-intent keywords"},
      {"channel": "LinkedIn Ads", "spend": 12000, "impressions": 180000, "clicks": 2400, "ctr": 1.3, "conversions": 68, "conversion_value": 29240, "cpa": 176.47, "roas": 2.4, "notes": "Higher quality leads but expensive"},
      {"channel": "Facebook/Instagram", "spend": 9500, "impressions": 890000, "clicks": 6200, "ctr": 0.7, "conversions": 42, "conversion_value": 18060, "cpa": 226.19, "roas": 1.9, "notes": "CTR has been declining month over month, same creative running since campaign start"},
      {"channel": "Content/SEO", "spend": 4200, "conversions": 89, "conversion_value": 38270, "cpa": 47.19, "roas": 9.1, "notes": "Organic blog traffic converting well, mostly from comparison pages"}
    ]
  }'
```

**Required:** `campaign_name`, `channels`

---

## Underperformance diagnosis

Claude doesn't just flag "Facebook is underperforming" — it diagnoses why based on the data provided. Declining CTR with unchanged creative points to creative fatigue. High CTR but low conversion points to landing page or offer mismatch. Each diagnosis includes a specific test to run to confirm the hypothesis.

---

## Budget reallocation

Reallocation recommendations are specific: "Move $4,000/month from Facebook/Instagram to Content/SEO" with the rationale (SEO's $47 CPA vs Facebook's $226 CPA) and expected impact (roughly how many more conversions this could generate at the blended rate).

---

## Blended vs channel-level

The blended metrics section shows overall campaign performance against target — useful when individual channels play different funnel roles (some channels are meant to have higher CPA because they reach higher-intent audiences).

---

## Limitations

- Analysis is based on the metrics you provide. For attribution-sensitive businesses, note that this doesn't account for cross-channel assist conversions — pass `conversion_value` that reflects your actual attribution model.
- Up to 15 channels per call, sufficient for most multi-channel campaigns.

---

## License

MIT.
