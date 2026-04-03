# FLOOWBOX - Legal Contract Analyzer Agent

Reading a 30-page contract thoroughly takes hours. Missing one clause can cost thousands. This workflow extracts every critical element, flags red flags, and writes a plain-English summary — in under 2 minutes.

## What it does

Send any contract URL. Jina AI fetches the full text. GPT-4o performs a comprehensive analysis: extracts parties, key dates, payment terms, contract value, termination clauses, liability caps, IP ownership, non-compete terms, and governing law. Identifies red flags with severity levels. Lists missing standard clauses. High-risk contracts fire an urgent Slack alert. A second GPT-4o pass writes a plain-English review report with a clear recommendation — SIGN AS IS, NEGOTIATE FIRST, or DO NOT SIGN.

## Tools Used
- **Orchestration:** n8n
- **Content Extraction:** Jina AI
- **Analysis:** OpenAI GPT-4o (Pass 1 — clause extraction)
- **Report Writing:** OpenAI GPT-4o (Pass 2 — plain English)
- **Alerts:** Slack (#legal)
- **Storage:** Notion
- **Trigger:** Webhook

## What gets extracted

```json
{
  "contract_summary": "Service agreement for 12-month automation consulting...",
  "payment_terms": "Net 30, monthly invoicing",
  "liability_caps": "Capped at 3 months contract value",
  "ip_ownership": "All deliverables become client property on full payment",
  "red_flags": [
    {
      "clause": "Unlimited revisions",
      "concern": "No revision limit defined — scope creep risk",
      "severity": "high"
    }
  ],
  "missing_standard_clauses": ["Dispute resolution", "Force majeure"],
  "negotiation_points": ["Add revision cap", "Clarify IP for pre-existing materials"],
  "overall_risk": "medium"
}
```

## Why I built this

FLOOWBOX reviews 3-5 contracts per month — client agreements, vendor contracts, partnership terms. I was spending 2-3 hours per contract on initial review. This does the first pass in 90 seconds and I only spend time on what actually needs human judgment.

## Important disclaimer

AI contract analysis is a first-pass review tool — not a substitute for qualified legal advice. Always verify with an actual lawyer for binding decisions.

## Setup

1. Jina AI API key
2. OpenAI API key
3. Slack Bot Token + #legal channel
4. Notion integration + DB ID
