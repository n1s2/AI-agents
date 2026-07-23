# policy-document-generator

Company policies written from scratch take hours and often end up too vague to be useful ("employees should behave professionally") or too rigid to accommodate real situations. This generates a complete, plain-language policy document with rationale for each rule, concrete examples, exception processes, enforcement details, and implementation notes for rollout.

---

## What it does

Takes policy name, type, company name, jurisdiction, specific requirements, existing practices, and known exceptions. Claude generates:

- Policy title, version, effective date, next review date, and owner
- **Purpose** — why this policy exists (rationale, not just scope)
- **Scope** — who it applies to and any exclusions
- **Definitions** — key terms defined plainly
- **Policy statements** — each section with: the rule (specific, not vague), concrete examples, and the rationale for this specific rule
- **Exceptions process** — how to request and approve exceptions
- **Enforcement** — what happens if the policy is violated
- **Employee responsibilities** and **manager responsibilities** separately
- **Related policies** that interact with this one
- **Implementation notes** — what HR/management should do to roll it out
- **Legal review flags** — specific items that need legal counsel before publishing

HTML formatted as a clean policy document with serif type, numbered sections, and legal flags highlighted in red.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-policy \
  -H "Content-Type: application/json" \
  -d '{
    "policy_name": "Remote Work Policy",
    "policy_type": "remote_work",
    "company_name": "Flowdesk Inc.",
    "company_size": "45 employees",
    "industry": "B2B SaaS",
    "jurisdiction": "US",
    "effective_date": "2025-07-01",
    "review_cycle": "annual",
    "approver_name": "Sarah Chen, COO",
    "specific_requirements": "We are fully remote with no office. Team is distributed across US time zones (PT to ET). Core hours 10am-3pm PT for synchronous work. Company provides $1,000 home office stipend for new employees and $500/year for equipment. Internet reimbursement up to $80/month. Employees expected to be reachable during core hours and attend all-hands on Thursdays.",
    "existing_practices": "Most of the team already works async with 24-hour response expectation on Slack outside core hours. Managers currently do weekly 1:1s via Zoom. We use Flowdesk for task tracking and Notion for documentation.",
    "exceptions": "Executives may have different travel and presence expectations. Customer-facing roles may have adjusted core hours for key accounts in other time zones.",
    "reply_email": "hr@flowdesk.com"
  }'
```

**Required:** `policy_name`, `policy_type`, `company_name`

---

## Policy types

`hr`, `security`, `acceptable_use`, `remote_work`, `data_privacy`, `expenses`, `code_of_conduct`, `social_media`, `incident_response`, `vendor_management`, `travel`, `pto`, `parental_leave`

Each type generates type-appropriate sections. A security policy covers access controls, incident reporting, and device requirements. A PTO policy covers accrual, carryover, and blackout periods. A code of conduct covers prohibited behaviors, reporting mechanisms, and non-retaliation.

---

## Legal review flags

Claude always includes a `legal_review_flags` list of specific items that need legal review before publishing — things like at-will employment language, FMLA interaction, state-specific requirements, or data retention commitments. These appear highlighted in red at the bottom of the HTML doc. This isn't legal advice — it's a checklist of what to have a lawyer look at.

---

## Limitations

- This is a first draft. All policies should be reviewed by qualified legal counsel before publishing, especially in areas involving employment law, data privacy (GDPR, CCPA), or regulated industries.
- Jurisdiction is used to calibrate the policy (US vs EU vs UK), but country-specific legal requirements vary by state/province. Pass jurisdiction as specifically as possible ("California, US" vs just "US") for better calibration.

---

## License

MIT.
