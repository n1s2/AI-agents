# FLOOWBOX - Document Q&A Agent (PDF and DOCX)

Send any document and ask it questions. The agent answers strictly from the document content — no hallucination, with exact quotes and confidence scores.

## What it does

Accepts a document URL (PDF, DOCX, web page, or public Google Doc) and a question. Jina AI extracts the full text. GPT-4o answers strictly from the document — quoting exact passages, rating its confidence, and flagging when the answer is not in the document. Returns follow-up question suggestions to guide the next query. All sessions log to Airtable.

## Tools Used
- **Orchestration:** n8n
- **Content Extraction:** Jina AI
- **QA Engine:** OpenAI GPT-4o
- **Logging:** Airtable
- **Trigger:** Webhook

## Flow

```
POST: {file_url, question, session_id}
  → Extract file metadata
  → Jina AI: fetch full document text
  → Truncate if over 3000 words (context limit management)
  → GPT-4o: answer from document only
  → Return: answer + quotes + confidence + follow-ups
  → Log session to Airtable
```

## Response format

```json
{
  "answer": "The payment terms are Net 30 from invoice date...",
  "quotes": ["Payment is due within thirty (30) days of invoice date"],
  "confidence": "high",
  "not_found": false,
  "follow_up": [
    "What are the late payment penalties?",
    "Are there any early payment discounts?"
  ]
}
```

## Use cases built for clients

- Contract review: "What are the termination clauses?"
- Policy documents: "What is the leave encashment policy?"
- Technical docs: "How do I authenticate with the API?"
- Research papers: "What methodology did they use?"
- RFPs: "What are the submission requirements?"

## Why I built this

A legal services client needed their team to quickly extract specific clauses from 50+ page contracts without reading the whole document every time. This reduced per-contract review time from 45 minutes to under 2 minutes for specific clause lookups.

## Setup

1. Jina AI API key
2. OpenAI API key
3. Airtable base + Document QA Log table
