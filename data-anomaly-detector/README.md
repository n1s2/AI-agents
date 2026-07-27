# data-anomaly-detector

Running analysis on bad data produces confidently wrong insights. Missing values, outliers, impossible values (a user who signed up before the product existed), format inconsistencies, and duplicates all corrupt downstream analysis if not caught first. This profiles your dataset, auto-computes column statistics, and uses Claude to identify anomalies, classify their severity, and recommend specific actions.

---

## What it does

Takes a dataset as a JSON array (up to 500 rows). The validator auto-computes column statistics: row count, null count, unique count, min/max/mean/stddev for numeric columns, and sample values. Claude then analyzes and returns:

- **Dataset health** — good/fair/poor/critical with summary
- **Fitness for use** — verdict on whether this data is ready to use as-is
- **Anomalies** — each with: column, type (outlier/error/missing/format_inconsistency/duplicate/impossible_value/suspicious_pattern), severity (critical/high/medium/low), description, affected row count, example values that triggered it, and recommendation
- **Column quality scores** — per-column 1–10 quality score with specific issues
- **Recommended actions** — each with priority (immediate/before_analysis/nice_to_have) and rationale
- **Patterns observed** — interesting patterns that may or may not be anomalies

HTML report with anomaly cards (color-coded by severity), column quality bars, and action priority badges.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/detect-anomalies \
  -H "Content-Type: application/json" \
  -d '{
    "dataset_name": "Q1 2025 Customer Revenue",
    "data_description": "Monthly revenue per customer from our billing system. Each row is one customer-month. Expected: all amounts positive, all dates in 2025, no duplicate customer-month combinations.",
    "expected_patterns": "Revenue should be between $50 and $50,000. All dates should be Jan-Mar 2025. Customer IDs should be UUIDs.",
    "known_issues": "We know there are some test accounts in the data with IDs starting with TEST-",
    "reply_email": "data@flowdesk.com",
    "data": [
      {"customer_id": "a1b2c3d4", "month": "2025-01", "revenue": 1200, "plan": "business", "country": "US"},
      {"customer_id": "e5f6g7h8", "month": "2025-01", "revenue": -50, "plan": "starter", "country": "US"},
      {"customer_id": "TEST-001", "month": "2025-01", "revenue": 0, "plan": "business", "country": "US"},
      {"customer_id": "i9j1k2l3", "month": "2025-01", "revenue": 250000, "plan": "starter", "country": "GB"},
      {"customer_id": "a1b2c3d4", "month": "2025-01", "revenue": 1200, "plan": "business", "country": "US"},
      {"customer_id": "m4n5o6p7", "month": "2024-12", "revenue": 800, "plan": "business", "country": "DE"}
    ]
  }'
```

**Required:** `dataset_name`, `data`

---

## What the validator computes automatically

Before Claude sees the data, the n8n Code node auto-profiles each column:
- `nullCount` — how many rows are missing this value
- `uniqueCount` — cardinality
- `isNumeric` — whether 80%+ of values parse as numbers
- `min`, `max`, `mean`, `stddev` — for numeric columns
- `sampleValues` — first 5 values

This means Claude gets statistical context that helps it distinguish "this value is 4 standard deviations from the mean" from "this value is slightly unusual."

---

## Anomaly types

| Type | What it means |
|---|---|
| `outlier` | Statistically unusual value — may be real |
| `error` | Almost certainly wrong (negative revenue, impossible date) |
| `missing` | Null or empty values in a field that should be populated |
| `format_inconsistency` | Same field has mixed formats (2025-01-01 vs January 2025) |
| `duplicate` | Same record appears more than once |
| `impossible_value` | Value that can't be real (signup before product existed) |
| `suspicious_pattern` | Pattern that warrants investigation (all round numbers, clustered at thresholds) |

---

## Row limit

500 rows maximum per call. For larger datasets, sample first (random or stratified) or run the agent on subsets. For time-series data, running on recent data catches regressions that whole-dataset averages would hide.

---

## Limitations

- Claude sees the first 50 rows of raw data plus computed statistics for all rows. For datasets with anomalies concentrated in later rows, the stats will surface them even if the raw rows don't show them.
- This detects data quality issues — it doesn't fix them. Use the recommendations to guide your cleaning process.

---

## License

MIT.
