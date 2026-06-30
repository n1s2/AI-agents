# data-quality-auditor

Bad data causes bad analysis, broken pipelines, and wrong decisions — often silently. Most teams only discover data quality issues when something downstream breaks. This audits a dataset proactively: profiles every field for null rates, unique values, and data types; flags issues with severity ratings; generates an ordered cleaning checklist; and assesses whether the data is actually ready to use for its intended purpose.

---

## What it does

1. Accepts up to 200 records as a JSON array, plus dataset name, description, business context, and optional expected fields list
2. Auto-profiles all fields: null rate, unique count, sample values, numeric range detection
3. Detects missing expected fields
4. Claude audits for missing/null values, duplicate risk, inconsistent formats, wrong data types, outliers, referential integrity, encoding/truncation issues
5. Returns: overall quality score (1–10) and label, issue list with severity and specific fix, per-field quality rating, ordered cleaning steps, positive findings, fitness for purpose assessment
6. Builds HTML report with score bar, issue cards, field quality table
7. Emails if `reply_email` provided

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Setup

**Env vars:** `FROM_EMAIL`
**Credentials:** Anthropic API (LangChain node), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/audit-data-quality \
  -H "Content-Type: application/json" \
  -d '{
    "dataset_name": "Q1 Customer Export",
    "dataset_description": "CRM export of all customers who signed up in Q1 2025",
    "business_context": "Needed for quarterly revenue analysis and email segmentation.",
    "expected_fields": ["customer_id", "email", "company", "plan", "mrr", "signup_date", "country"],
    "reply_email": "analytics@company.com",
    "records": [
      {"customer_id": "C001", "email": "alice@acme.com", "company": "Acme Corp", "plan": "Pro", "mrr": 299, "signup_date": "2025-01-14", "country": "US"},
      {"customer_id": "C002", "email": "bob@globex", "company": "Globex", "plan": "starter", "mrr": "99", "signup_date": "Jan 22 2025", "country": ""},
      {"customer_id": "C003", "email": "carol@initech.com", "company": null, "plan": "Pro", "mrr": 299, "signup_date": "2025-02-01", "country": "UK"},
      {"customer_id": "C001", "email": "alice@acme.com", "company": "Acme Corp", "plan": "Pro", "mrr": 299, "signup_date": "2025-01-14", "country": "US"}
    ]
  }'
```

In this example, Claude flags duplicate C001, malformed email for C002, inconsistent date formats, mixed string/number MRR types, null company for C003, and inconsistent country codes.

**Required:** `dataset_name`, `records` (array, min 2)

---

## Record limit

Up to 200 records. For larger datasets, pass a representative sample — audit is statistical and pattern-based, so 50–100 records usually surfaces structural issues.

---

## Severity levels

**Critical** — will break things: duplicate keys, missing required fields, wrong data types causing calculation errors
**Significant** — will silently produce wrong results: high null rates, inconsistent formats affecting grouping, skewing outliers
**Minor** — worth cleaning, not urgent: low null rates, whitespace, cosmetic formatting

---

## Limitations

- Profiling reflects the submitted sample, not the full dataset.
- Detects structural/format issues, not semantic correctness (e.g., whether a revenue figure is plausible requires domain knowledge).

---

## License

MIT.
