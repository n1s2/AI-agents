# legal-document-summarizer

Most people sign contracts they don't fully understand because reading legal documents is slow, uncomfortable, and uncertain. You can spend an hour with a contract and still not be sure what you agreed to.

This takes a legal document and produces a plain-language summary calibrated to who's reading it: what the document does, what you're agreeing to, what rights you gain, what obligations you take on, specific red flags with context on why they're unusual, what's typically in this type of document that's missing here, and a list of specific questions to ask your lawyer before signing.

It's not legal advice — it says so clearly. It's the 15-minute prep work that makes a 30-minute lawyer conversation actually useful.

---

## What it does

1. Accepts a POST: document text (up to 15,000 characters), document type, reader's party role, audience, jurisdiction, specific concerns
2. Claude summarizes at `temperature: 0.2` for precision:
   - Document overview (what it is, who the parties are)
   - Plain English summary for the reader's position
   - Key provisions: each with plain English explanation, practical implications, severity rating (neutral / noteworthy / concerning / critical)
   - Obligations on the reader
   - Rights granted to the reader
   - Red flags: unusual or one-sided clauses with comparison to what's typical
   - Missing protections: things normally present in this document type that are absent
   - Termination summary
   - Liability and indemnification summary
   - Key dates and deadlines
   - Questions to ask a lawyer (specific, based on this document's content)
   - Overall risk assessment: low / moderate / elevated / high
3. Builds an HTML summary with color-coded severity indicators
4. Emails if `reply_email` provided
5. Returns full JSON

---

## Stack

- **n8n** — webhook + workflow
- **Anthropic Claude** (claude-sonnet-4-20250514) — summarization at low temperature
- **SMTP** — optional email delivery

---

## Setup

### 1. Environment variables

```
FROM_EMAIL=legal@yourdomain.com
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** (optional)

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/summarize-legal \
  -H "Content-Type: application/json" \
  -d '{
    "document_type": "service_agreement",
    "document_title": "Master Services Agreement — Acme Corp",
    "party_role": "vendor / service provider",
    "audience": "founder",
    "jurisdiction": "Delaware, USA",
    "specific_concerns": "We are a small company and want to understand our liability exposure and IP ownership terms carefully.",
    "reply_email": "legal@ourcompany.com",
    "document_text": "[paste the full contract text here]"
  }'
```

**Required:** `document_text`, `document_type`

---

## Document types

`contract`, `nda`, `terms_of_service`, `privacy_policy`, `employment_agreement`, `lease`, `software_license`, `partnership_agreement`, `service_agreement`, `settlement`, `other`

The document type shapes what Claude looks for. An NDA summary emphasizes scope of confidentiality, exceptions, duration, and permitted disclosures. A software license summary emphasizes usage rights, restrictions, sublicensing, and IP assignment.

---

## Audience calibration

| Audience | Emphasis |
|---|---|
| `founder` | Business risk, IP ownership, liability caps, investor-related terms |
| `non_lawyer_business` | Practical obligations, financial exposure, exit terms |
| `employee` | Compensation terms, IP assignment, non-compete, termination |
| `technical_team` | Data handling, API terms, acceptable use, SLAs |
| `general` | Balanced overview, no role-specific emphasis |

---

## Severity ratings

Each key provision gets a severity rating:

| Severity | Meaning |
|---|---|
| `neutral` | Standard clause, nothing unusual |
| `noteworthy` | Worth understanding, not necessarily concerning |
| `concerning` | One-sided or unusual — worth discussion with lawyer |
| `critical` | Significant risk — should not be signed without lawyer review and likely negotiation |

---

## Red flags

Claude flags clauses that are unusual given this document type — things like:
- Automatic renewal with short cancellation windows
- Unilateral right to modify terms
- Broad IP assignment that covers pre-existing work
- Indemnification that's one-sided or uncapped
- Jurisdiction clauses that require dispute resolution in an inconvenient forum
- Arbitration clauses that waive right to jury trial

Each red flag includes what the clause typically looks like in a balanced agreement, so the reader understands what they're accepting vs what's normal.

---

## Missing protections

For each document type, there are provisions that typically should be present. If they're absent, Claude lists them. For an NDA missing a "residuals" carveout, for a service agreement missing a liability cap, for an employment agreement missing a severance provision — these absences matter.

---

## Questions for lawyer

The `questions_for_lawyer` section is the most actionable part. Rather than "talk to a lawyer about this contract," it gives specific questions derived from the actual content: "Section 12.3 assigns all IP to the client — does this include software developed on our own time or tools? How would we protect our existing codebase?" These questions make a lawyer consultation more efficient and less expensive.

---

## Document length limit

The webhook accepts up to 15,000 characters of document text — roughly 10–12 pages. For longer documents:
- Submit the most critical sections (payment, IP, termination, liability)
- Or split by section and summarize each independently
- The `specific_concerns` field helps Claude prioritize what to focus on when the document is complex

---

## Low temperature setting

This agent runs at `temperature: 0.2` — very low for maximum accuracy and precision. Legal document summarization requires staying close to what the document actually says, not interpreting or extrapolating. The low temperature minimizes creative interpretation.

---

## Limitations

This is not legal advice. The summary:
- May miss nuanced legal interpretations that only a lawyer familiar with the jurisdiction would catch
- Cannot assess whether clauses are enforceable under specific local law
- Does not replace lawyer review for high-stakes documents (large contracts, IPO-related documents, settlement agreements, etc.)
- May not catch every unusual clause in very long or complex documents

For documents that are business-critical or involve significant liability, always have a lawyer review the original document.

---

## Ideas

- [ ] Clause comparison: submit two versions of the same document, highlight what changed
- [ ] Template library: flag which clauses in this document are "market standard" vs custom
- [ ] Redline suggestions: given a red flag, suggest specific alternative language to negotiate
- [ ] Document Q&A: after the summary, ask specific questions about the document ("Does this NDA cover our source code?")

---

## License

MIT. Not legal advice.
