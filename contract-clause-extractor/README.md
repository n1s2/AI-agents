# contract-clause-extractor

Reading a 20-page vendor contract to find the auto-renewal clause, the liability cap, and anything that limits your ability to terminate takes an hour and still misses things. This extracts specific clauses from a contract, translates them to plain English, rates risk level, identifies what favors the buyer vs seller, flags missing clauses that should be there, surfaces key dates and triggers, and suggests specific redline changes — with a clear note that legal review is still required.

---

## What it does

Takes contract text (up to 10,000 characters), contract type, parties, and reviewer perspective. Claude extracts and analyzes:

- **Contract summary** — 3–4 sentences: parties, scope, key terms at a glance
- **Extracted clauses** — each with: clause type, section reference, raw text, plain English explanation, risk level (standard/watch/high_risk/critical), risk notes, and who it favors (buyer/seller/neutral)
- **Missing clauses** — clauses that should be present for this contract type but aren't, with risk of absence
- **Key dates and triggers** — events, dates or trigger conditions, and required action
- **High-risk summary** — list of terms to escalate for legal review
- **Redline suggestions** — current language vs suggested change with rationale
- **Legal review required** flag with specific notes on what needs counsel attention

HTML report with clause cards color-coded by risk level, missing clauses highlighted in amber, and redlines with current/suggested side-by-side.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/extract-contract-clauses \
  -H "Content-Type: application/json" \
  -d '{
    "contract_id": "VENDOR-2025-047",
    "contract_type": "saas_subscription",
    "parties": ["Flowdesk Inc.", "Acme Analytics Ltd."],
    "effective_date": "2025-07-01",
    "reviewer_perspective": "buyer",
    "extraction_focus": ["payment_terms", "termination", "liability", "data_privacy", "renewal"],
    "flag_high_risk": true,
    "reply_email": "legal@flowdesk.com",
    "contract_text": "[Paste contract text here — up to 10,000 characters]"
  }'
```

**Required:** `contract_text`, `contract_type`

---

## Contract types

`saas_subscription`, `employment`, `nda`, `vendor_services`, `partnership`, `lease`, `consulting`, `other`

Type calibrates what missing clauses to flag. A SaaS subscription should have a data processing agreement, SLA, and clear termination clause. An NDA should have a definition of confidential information and a return-of-materials clause. An employment contract should have at-will language (in applicable jurisdictions) and IP assignment.

---

## Extraction focus

`payment_terms`, `termination`, `liability`, `ip_ownership`, `data_privacy`, `non_compete`, `renewal`, `sla`, `indemnification`, `governing_law`, `all`

Pass `["all"]` to extract everything Claude finds, or pass a specific list to focus on the clauses you care about most.

---

## Risk levels

| Level | Meaning |
|---|---|
| `standard` | Typical for this contract type, no unusual terms |
| `watch` | Worth noting, not immediately concerning |
| `high_risk` | Unusually one-sided or potentially problematic — review carefully |
| `critical` | Should not sign without legal review and likely negotiation |

---

## Important disclaimer

This agent extracts and explains contract text — it is not legal advice. The `legal_review_required` flag is almost always true. Use this for initial triage and briefing, not as a substitute for qualified legal counsel.

---

## Limitations

- Contract text is capped at 10,000 characters (~1,500 words). Long contracts should be split by section and run in multiple calls, or the most critical sections should be passed.
- Extraction accuracy depends on contract clarity. Vague or highly templated contracts with heavy cross-references may produce incomplete extractions.

---

## License

MIT.
