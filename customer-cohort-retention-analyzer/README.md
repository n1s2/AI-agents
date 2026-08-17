# customer-cohort-retention-analyzer

A cohort retention table full of percentages doesn't tell you whether things are getting better or worse, or why. This renders a color-coded retention heatmap and has Claude analyze whether the curve is flattening (good — indicates a sticky core) or continuing to decay (concerning), which cohorts are over/underperforming, and what segment or channel factors explain the differences.

---

## What it does

Takes an array of cohorts, each with a label, starting customer count, retention percentage by period, segment, and acquisition channel. Claude produces:

- **Retention summary** and **trend assessment** — improving/stable/declining/mixed
- **Curve shape analysis** — whether retention is flattening at a stable floor or continuing to decay, and at what period
- **Cohort comparison** — each cohort rated outperforming/average/underperforming with notable pattern
- **Segment or channel insights** — which factors correlate with better/worse retention
- **Critical drop-off period** — where the biggest single-period drop happens, with hypothesis for why
- **Benchmark context** — how this compares to typical SaaS retention profiles, with appropriate caveats
- **Recommendations** — prioritized, with rationale
- **Data quality notes** — honest observations about sample size or gaps that limit confidence

HTML report with a color-coded retention heatmap table (green=high retention, red=low), plus analysis sections below.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/analyze-cohort-retention \
  -H "Content-Type: application/json" \
  -d '{
    "product_name": "Flowdesk",
    "period_type": "month",
    "business_context": "B2B SaaS, self-serve + sales-assisted for larger accounts. Product changed significantly in March 2025 with new onboarding flow.",
    "reply_email": "product@flowdesk.com",
    "cohorts": [
      {"label": "Jan 2025", "starting_customers": 85, "segment": "SMB", "acquisition_channel": "organic", "retention_by_period": [100, 78, 65, 58, 52, 49]},
      {"label": "Feb 2025", "starting_customers": 92, "segment": "SMB", "acquisition_channel": "paid", "retention_by_period": [100, 71, 55, 47, 41]},
      {"label": "Mar 2025", "starting_customers": 78, "segment": "SMB", "acquisition_channel": "organic", "retention_by_period": [100, 85, 76, 71]},
      {"label": "Apr 2025", "starting_customers": 104, "segment": "SMB", "acquisition_channel": "organic", "retention_by_period": [100, 88, 79]},
      {"label": "May 2025", "starting_customers": 96, "segment": "mid_market", "acquisition_channel": "sales", "retention_by_period": [100, 91]}
    ]
  }'
```

**Required:** `cohorts`

---

## Retention format

Each cohort's `retention_by_period` is an array where index 0 is period 0 (typically 100%, the starting point) and each subsequent index is the percentage of the original cohort still active at that period. Cohorts don't need equal length arrays — newer cohorts naturally have fewer periods of data, and the heatmap handles this gracefully with empty cells for missing future periods.

---

## Curve flattening vs decay

This is the single most important read in cohort analysis. A curve that drops sharply then flattens (e.g., 100% → 60% → 45% → 44% → 43%) indicates a stable core of engaged users — expected and healthy. A curve that keeps declining without ever flattening (100% → 60% → 45% → 30% → 15%) indicates the product isn't creating durable habit, which is a much more serious problem. Claude's `curve_shape_analysis` explicitly calls out which pattern is happening.

---

## Before/after comparison

In the example above, the March cohort onward shows notably better retention — this coincides with the described product change (new onboarding flow). Claude will surface this as a positive signal in `cohort_comparison` and `segment_or_channel_insights` if the pattern holds.

---

## Limitations

- Up to 24 cohorts per call. For longer histories, aggregate by quarter instead of month, or run separately by segment.
- Analysis quality depends on cohort size — small starting cohorts (n<20) produce noisy percentages that Claude will flag in `data_quality_notes` as lower confidence.

---

## License

MIT.
