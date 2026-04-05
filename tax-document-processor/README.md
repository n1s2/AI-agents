# FLOOWBOX - Tax Document Processor Agent

Tax documents are full of numbers that need to be cross-referenced and calculated. This workflow processes any tax document — Form 16, salary slips, capital gains statements — and produces a clear summary with your exact tax position in minutes.

## What it does

Upload any tax document URL. Jina AI extracts the full text. GPT-4o performs a two-pass analysis: first extracting all income sources, deductions, TDS deducted, and computing the tax position. Second pass identifies additional deductions that could still be claimed and provides next-year tax saving tips. Results save to Notion and an email report goes out.

## Tools Used
- **Orchestration:** n8n
- **Content Extraction:** Jina AI
- **Pass 1 Analysis:** OpenAI GPT-4o (data extraction)
- **Pass 2 Advisory:** OpenAI GPT-4o (savings opportunities)
- **Storage:** Notion
- **Report:** Email
- **Trigger:** Webhook

## Flow

```
POST: {url, type, year, country}
  → Fetch document via Jina AI
  → GPT-4o Pass 1: extract all income, deductions, TDS
  → GPT-4o Pass 2: calculate position + identify savings
  → Save to Notion
  → Send email with full breakdown
  → Return JSON response
```

## Documents supported

- Form 16 (India) — salary TDS certificate
- Capital gains statements
- Interest certificates (Form 16A)
- Salary slips (annual summary)
- Business P&L statements
- Any standard tax document in text/PDF format

## Example output

```json
{
  "income_sources": [
    {"source": "Salary — Employer Ltd", "amount": 1200000, "type": "salary"}
  ],
  "deductions": [
    {"section": "80C", "description": "PF + LIC", "amount": 150000},
    {"section": "80D", "description": "Health insurance", "amount": 25000}
  ],
  "taxable_income": 1025000,
  "tds_deducted": 98000,
  "refund_due": 12500,
  "additional_deductions_possible": [
    {"section": "80CCD(1B)", "max_amount": 50000, "description": "NPS additional contribution"}
  ]
}
```

## Important disclaimer

AI tax analysis is for informational purposes only. Always verify calculations and file with the guidance of a qualified Chartered Accountant.

## Setup

1. Jina AI API key
2. OpenAI API key
3. Notion integration + DB ID
4. SMTP credentials
