# contract-clause-explainer

Most people sign contracts they don't fully understand because hiring a lawyer for a freelance agreement or apartment lease feels disproportionate, and asking the other party to explain their own contract to you feels awkward.

This doesn't replace a lawyer. It says that clearly once, at the top, then gets useful. You paste the contract text, tell it your role and the contract type, and it breaks down every clause in plain English: what it actually says, what it means in practice for you, whether it's standard or worth pushing back on, what's missing that should be there, and what questions to ask before signing.

The clause-by-clause section is risk-rated: standard, watch, negotiate, or red flag. Red flags come with a count in the email subject line so you know before you open it whether something needs immediate attention.

---

## What it does

1. Accepts a POST: contract text (up to ~12,000 characters), contract type, your role in the contract, jurisdiction, any specific clauses to focus on
2. Sends to Claude which produces a full analysis:
   - Plain-English summary of what the contract does
   - Your obligations vs their obligations
   - Every identifiable clause with: plain-English explanation, real-world implication, risk level (standard/watch/negotiate/red_flag)
   - Red flags list
   - Specific negotiation suggestions with what to ask for
   - Missing protections that should normally be in this type of contract
   - Questions to ask before signing
3. Formats into a clean sectioned HTML report color-coded by risk level
4. Emails the report if `reply_email` is provided
5. Returns full JSON in the webhook response

---

## Stack

- **n8n** — webhook + workflow
- **Anthropic Claude** (claude-opus-4-5) — contract analysis
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
curl -X POST https://your-n8n.com/webhook/explain-contract \
  -H "Content-Type: application/json" \
  -d '{
    "contract_text": "1. SERVICES. Contractor shall provide software development services as described in Schedule A...\n2. PAYMENT. Client shall pay Contractor $150/hr within 30 days of invoice...\n3. IP OWNERSHIP. All work product created under this Agreement shall be the sole and exclusive property of Client, including all intellectual property rights therein...\n4. NON-COMPETE. Contractor agrees not to work with any client in the same industry for 12 months following termination...",
    "contract_type": "freelance",
    "party_role": "the contractor (me)",
    "jurisdiction": "California, USA",
    "specific_clauses": "IP ownership and non-compete are my main concerns",
    "reply_email": "me@email.com"
  }'
```

**Required:** `contract_text`

**Strongly recommended:** `contract_type`, `party_role`, `specific_clauses` — these significantly improve the relevance of the analysis

---

## Contract types

`employment`, `freelance`, `saas`, `nda`, `lease`, `partnership`, `service`, `vendor`, `consulting`, `other`

The type helps Claude understand what's normal for this kind of agreement. An aggressive IP clause in a freelance contract is flagged differently than the same clause in an employment contract.

---

## The `party_role` field

Tell Claude which side you're on. "The contractor", "the tenant", "the buyer", "the service provider". This determines whose obligations get listed under "your obligations" and shapes how risk is assessed. A clause that's unfavorable for one party may be entirely normal from the other's perspective.

---

## Risk levels explained

| Level | Meaning |
|---|---|
| **Standard** | Normal for this contract type. No action needed. |
| **Watch** | Worth understanding fully. Not necessarily a problem but pay attention. |
| **Negotiate** | This clause is worth pushing back on. Claude suggests what to ask for. |
| **Red Flag** | Unusual, one-sided, or potentially harmful. Get this changed or get advice. |

---

## Contract size limits

The webhook accepts up to 12,000 characters of contract text — roughly 8–12 pages of dense legal text. For longer contracts:
- Paste the most important sections (especially anything flagged in `specific_clauses`)
- Or split into multiple submissions by section
- Claude handles partial contracts fine — it analyses what it has and notes if context seems to be missing

---

## Building a self-service tool

Works well as a team tool for companies that receive lots of inbound contracts (vendor agreements, SaaS MSAs, NDAs). Embed a simple form in your internal tools wiki, paste the contract, get the analysis in under 30 seconds. Saves the "can you look at this NDA quickly" Slack messages to your legal team.

Could also be exposed as a public tool — add rate limiting and basic auth to the webhook if you go that route.

---

## What this is not

This is not legal advice and it says so clearly. Claude doesn't know your specific situation, prior agreements, or local law nuances. For anything involving significant money, long-term obligations, or employment terms, have a lawyer review the contract. This tool helps you understand what you're looking at and know which questions to ask — it's not a substitute for qualified legal counsel.

The analysis is good for:
- Understanding what you're agreeing to before the lawyer call
- Spotting obvious red flags in a standard freelance or SaaS agreement
- Knowing which clauses to specifically ask your lawyer about
- Preparing your negotiation points

---

## Limitations

- 12,000 character limit on contract text. Roughly 8 pages.
- Claude's knowledge of what's "standard" is based on general commercial practice. Jurisdiction-specific norms (employment law in Germany, tenancy law in New South Wales) may not be fully reflected. Always specify jurisdiction.
- Claude identifies missing protections based on common contract patterns. Genuinely unusual contracts may confuse the heuristics.
- The `negotiate_these` section suggests what to ask for but doesn't know the negotiating dynamics — whether the other party will budge is a separate question.

---

## Ideas

- [ ] PDF upload mode — parse contract PDF text before sending to Claude
- [ ] Side-by-side comparison — submit two versions of a contract, Claude marks what changed and whether changes are favorable
- [ ] Redline suggestions — Claude returns suggested alternative language for flagged clauses
- [ ] Team library — save analyses to a sheet for reference when similar contracts come up again

---

## License

MIT. Not legal advice.
