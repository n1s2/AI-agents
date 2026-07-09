# vendor-risk-assessor

Most vendor risk assessments are checkbox exercises — "do you have SOC 2?" followed by filing the certificate and moving on. Real vendor risk depends on what data they touch, how critical they are to operations, where their subprocessors are, whether you actually have the contractual protections you need, and what their public security history looks like. This does a structured risk assessment across five dimensions and searches for recent public incidents.

---

## What it does

Takes vendor details: name, category, service type, data access level, business criticality, spend, single-point-of-failure status, subprocessors, geographies, compliance frameworks in place, and contract/BAA status. Searches Tavily for recent security incidents or compliance issues involving the vendor. Claude produces:

- Overall risk score (1–10) and level (low/medium/high/critical)
- Risk dimensions with per-dimension scores and specific findings: operational, security, compliance, financial, concentration
- Critical gaps with remediation steps and priority (immediate/30 days/90 days)
- Positive controls (things that reduce risk)
- Due diligence checklist (specific documents/questions to obtain)
- Contract requirements (specific clauses to ensure are in place)
- Ongoing monitoring actions (periodic checks to run)
- Public intel summary from search results

---

## Stack

n8n, Tavily API, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Setup

**Env vars:** `TAVILY_API_KEY`, `FROM_EMAIL`

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/assess-vendor-risk \
  -H "Content-Type: application/json" \
  -d '{
    "vendor_name": "Segment",
    "vendor_category": "Customer data platform",
    "service_type": "saas",
    "vendor_description": "We use Segment to collect and route customer event data to our analytics tools and CRM",
    "annual_spend": 24000,
    "currency": "USD",
    "business_criticality": "high",
    "single_point_of_failure": false,
    "data_access_level": "sensitive_pii",
    "geographies_data_stored": ["US", "EU"],
    "vendor_subprocessors": "Twilio (parent), AWS, Google Cloud",
    "compliance_frameworks": ["SOC 2 Type II", "GDPR", "CCPA"],
    "contract_in_place": true,
    "has_baa": false,
    "has_security_questionnaire": true,
    "additional_context": "Segment receives full user event streams including email, user ID, and behavioral data. We process EU users so GDPR data processing agreement is required.",
    "reply_email": "security@company.com"
  }'
```

**Required:** `vendor_name`, `vendor_category`, `data_access_level`

---

## Data access levels

`none`, `public`, `internal`, `confidential`, `sensitive_pii`, `critical` — used to calibrate the security and compliance risk assessment. A vendor with `sensitive_pii` access gets a higher baseline risk and more stringent contract requirements than one with `internal` access only.

---

## Public intel search

Tavily searches for recent security breaches, data incidents, and compliance failures involving the vendor. This surfaces relevant history that affects the risk score — a vendor with a recent breach gets a higher security risk score and specific remediation items.

---

## Limitations

- Risk assessment is based on information you provide plus public search results. It doesn't access the vendor's actual security controls or internal documents.
- Legal and compliance determinations (e.g., whether a specific contract clause is sufficient) require qualified legal review. This agent gives specific guidance on what to include but doesn't substitute for legal counsel.

---

## License

MIT.
