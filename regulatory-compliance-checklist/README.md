# regulatory-compliance-checklist

"Are we GDPR compliant?" is not a yes/no question most teams can answer without a structured checklist. This searches for current information on the specified regulation, generates a categorized readiness checklist calibrated to your business context, flags high-risk gaps, and is explicit about what's operational implementation versus what needs actual legal counsel — because this is not a substitute for qualified legal review.

---

## What it does

Takes regulation name, business context, company size, industry, data types handled, geographic scope, and current measures in place. Searches Tavily for current information on the regulation. Claude produces:

- **Applicability assessment** — whether and how this regulation likely applies, with appropriate uncertainty language
- **Categories** — requirements grouped (e.g., Data Subject Rights, Consent Management, Security Measures), each item with: priority (must_have/should_have/best_practice), plain-language description, practical implementation notes for a small/mid company, and status assessment against your current measures (likely_met/partially_met/likely_gap/needs_legal_review)
- **High-risk gaps** — specific gaps with risk if unaddressed and suggested priority
- **Items requiring legal review** — explicitly flagged, not resolved by the checklist itself
- **Documentation needed** — specific documents or records typically required
- **Ongoing obligations** — things requiring continuous compliance activity, not one-time fixes
- **Disclaimer** — clear statement this is not legal advice

HTML report with a prominent disclaimer banner, category sections, and legal-review items separated out in purple.

---

## Stack

n8n, Tavily API, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-compliance-checklist \
  -H "Content-Type: application/json" \
  -d '{
    "regulation_name": "GDPR",
    "business_context": "B2B SaaS project management tool. We store customer data including names, emails, and task content. Some customers are based in the EU.",
    "company_size": "45 employees",
    "industry": "SaaS / project management",
    "data_types": ["names", "email addresses", "task content", "usage analytics", "IP addresses"],
    "geographic_scope": ["US headquartered", "EU customers", "UK customers"],
    "current_measures": "We have a privacy policy, use encrypted connections, data is hosted on AWS in US and EU regions. No formal data processing agreements with all subprocessors yet. No designated DPO.",
    "tech_stack": "AWS, PostgreSQL, Segment for analytics, Intercom for support",
    "reply_email": "legal@flowdesk.com"
  }'
```

**Required:** `regulation_name`, `business_context`

---

## Not legal advice

This is stated explicitly in every output and reinforced in the system prompt. The checklist is a structured starting point for a conversation with qualified counsel — it helps you understand the shape of the compliance landscape and prioritize what to bring to a lawyer, not a substitute for legal review.

---

## Status assessment against your current measures

If you pass `current_measures`, each checklist item gets assessed against what you've described — `likely_met`, `partially_met`, `likely_gap`, or `needs_legal_review`. This turns a generic checklist into something specific to your actual gaps, though the assessment is necessarily approximate given the information provided.

---

## High-risk gaps prioritized

Not every gap is equally urgent. The `high_risk_gaps` section surfaces the items where the risk of non-compliance is most significant, with a suggested priority (immediate/near_term/planned) — useful for deciding where to focus limited compliance resources first.

---

## Limitations

- This is generated from public research and general regulatory knowledge — it is not a substitute for jurisdiction-specific, business-specific legal counsel. Regulations change and have nuances that require professional interpretation.
- Applicability assessment is directional, not determinative. Whether a regulation actually applies to your business requires legal analysis of your specific facts.

---

## License

MIT.
