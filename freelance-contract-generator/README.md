# freelance-contract-generator

Most freelancers either work without a contract or use a template they found online that hasn't been looked at since 2018. Neither is good. A bad contract either leaves the freelancer exposed (no kill fee, vague IP terms, no revision limit) or makes clients uncomfortable (aggressive non-competes, punishing late fees).

This generates a proper freelance service contract calibrated to your situation: contract type, jurisdiction, payment structure, deliverables, IP clauses, NDA, and special terms. The output is plain English — clear enough to read without a law degree — with all the clauses that actually protect both parties. It also flags what should be reviewed by a lawyer before signing.

Not a substitute for legal counsel, but a much better starting point than a Google doc template.

---

## What it does

1. Accepts a POST: freelancer and client details, project description, deliverables, contract type, payment amount, payment schedule, dates, jurisdiction, revision rounds, optional NDA and non-compete, late fee rate, special terms
2. Claude drafts a complete service agreement with:
   - Parties
   - Services & Deliverables (specific scope)
   - Timeline & Milestones
   - Payment Terms (with kill fee mechanics)
   - Revisions & Change Orders
   - Intellectual Property
   - Independent Contractor Status
   - Confidentiality (+ NDA if requested)
   - Limitation of Liability
   - Termination
   - Dispute Resolution (jurisdiction-specific)
   - General Provisions
3. Returns a key terms summary (plain English)
4. Lists freelancer protections included in the contract
5. Lists client obligations
6. Flags specific clauses to review with a lawyer
7. Builds formatted HTML contract document
8. Emails if `reply_email` provided
9. Returns full JSON with all sections

---

## Stack

- **n8n** — webhook + workflow
- **Anthropic Claude** (claude-sonnet-4-20250514) — contract drafting
- **SMTP** — optional email delivery

---

## Setup

### 1. Environment variables

```
FROM_EMAIL=contracts@yourdomain.com
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** (optional)

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-contract \
  -H "Content-Type: application/json" \
  -d '{
    "freelancer_name": "Maya Osei",
    "freelancer_email": "maya@mayadesign.co",
    "freelancer_address": "42 Studio Lane, Portland, OR 97201",
    "freelancer_business_name": "Maya Osei Design",
    "client_name": "Brightfield Technologies Inc.",
    "client_email": "legal@brightfield.com",
    "client_address": "888 Market Street, San Francisco, CA 94102",
    "client_contact_name": "James Park, Head of Product",
    "project_description": "Brand identity redesign including logo, color palette, typography system, and brand guidelines document. Project covers research phase, concept development (3 directions), client review and refinement, and final delivery of all assets.",
    "deliverables": [
      "Logo in SVG, PNG, and PDF formats (primary + variations)",
      "Color palette with hex, RGB, and CMYK values",
      "Typography guidelines with licensed font recommendations",
      "Brand guidelines PDF (20-30 pages)",
      "Social media template kit (5 templates in Figma)"
    ],
    "contract_type": "milestone_based",
    "payment_amount": 8500,
    "currency": "USD",
    "payment_schedule": "30% on signing ($2,550), 30% on concept approval ($2,550), 40% on final delivery ($3,400)",
    "payment_terms_days": 14,
    "start_date": "2025-06-01",
    "end_date": "2025-08-15",
    "jurisdiction": "us_california",
    "revision_rounds": 3,
    "late_fee_percent": 1.5,
    "include_ip_clause": true,
    "include_nda": true,
    "include_non_compete": false,
    "special_terms": "Client will provide brand brief, competitor references, and stakeholder interview access within 5 business days of contract signing. Any delays caused by late client feedback extend the project timeline accordingly.",
    "reply_email": "maya@mayadesign.co"
  }'
```

**Required:** `freelancer_name`, `client_name`, `project_description`, `payment_amount`, `currency`

---

## Contract types

| Type | Structure |
|---|---|
| `fixed_price` | Single payment or simple split on completion |
| `hourly` | Rate, estimated hours, billing cadence |
| `retainer` | Monthly fee, hours included, rollover policy |
| `milestone_based` | Payment tied to specific deliverable approvals |
| `revenue_share` | Percentage-based compensation terms |

---

## Jurisdictions

| Code | Notes |
|---|---|
| `us_general` | US contract without state-specific provisions |
| `us_california` | Includes CA-specific provisions (IP work for hire, contractor classification) |
| `uk` | UK contract law, GDPR references, IR35 consideration |
| `eu` | EU contract law, GDPR compliance |
| `canada` | Canadian contract law |
| `australia` | Australian contract law |
| `international` | Neutral jurisdiction clause, suitable for cross-border work |

California requires specific language around IP assignment and contractor status — the contract includes this automatically when `us_california` is selected.

---

## Key clauses explained

**IP assignment** — By default, the client gets full IP ownership on receipt of final payment. Before that, all work is the freelancer's. This is standard and fair — it means the client can't use the work before paying.

**Kill fee** — If the client terminates mid-project, the freelancer keeps all payments made and receives a kill fee for work completed. The kill fee percentage is calculated from the project total and work completed. This is the clause most freelancers forget to include.

**Revision rounds** — Defined as specific numbered rounds, not "reasonable revisions." After the included rounds, additional revisions are billed at a day rate. This is the primary defense against scope creep.

**Change orders** — If the client requests work outside the defined scope, it requires a written change order with price and timeline before work begins.

**Independent contractor status** — Explicitly confirms the freelancer is not an employee. Required language in US and UK contracts.

---

## NDA option

`include_nda: true` adds a mutual NDA clause covering both parties' confidential information. Useful when the freelancer will have access to unreleased products, customer data, or proprietary processes. The NDA is mutual — both parties are bound.

---

## Non-compete option

`include_non_compete: false` by default. A non-compete restricts the freelancer from working with direct competitors for a period. These are:
- Increasingly unenforceable in many US states (California bans them outright)
- Harmful to freelancers who may have multiple clients in the same industry
- Rarely appropriate for project-based freelance work

Think carefully before enabling this, and check local enforceability.

---

## Special terms

The `special_terms` field is for anything not covered by the standard structure. Common uses:
- Client obligations (content delivery deadlines, feedback turnaround)
- Portfolio rights (may the freelancer show this work?)
- Technology transfer details
- Specific warranty limitations

---

## Review flags

The contract includes a "Review with a lawyer before signing" section that calls out clauses specific to your situation that warrant legal review — not generic advice but things in this particular contract that have legal nuance. For California contracts, this typically includes IP assignment language and contractor classification.

---

## Limitations

This is a starting point, not a finalized legal document. The output is a good-faith draft that covers standard freelance scenarios. It should be reviewed by a qualified attorney in your jurisdiction before use, particularly for:
- High-value contracts (>$25,000)
- Work involving regulated industries (health, finance, legal)
- International engagements with cross-border IP considerations
- Situations where employment classification is ambiguous

---

## Ideas

- [ ] PDF export: convert the HTML contract to a signed PDF via a PDF generation API
- [ ] DocuSign integration: send for e-signature automatically after generation
- [ ] Contract library: save all generated contracts to Google Drive with consistent naming
- [ ] Invoice generator: companion agent that generates invoices matching the payment schedule from an accepted contract

---

## License

MIT. Not legal advice.
