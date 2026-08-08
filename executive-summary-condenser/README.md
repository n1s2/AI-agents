# executive-summary-condenser

A 20-page research report or technical spec doesn't get read by executives — it gets skimmed, or worse, ignored entirely because there's no time. This condenses long documents into a genuine executive summary: bottom line first, quantified where the source is, honest about what's uncertain, and explicit about what got cut so the reader knows when to go back to the source.

---

## What it does

Takes a source document (up to 15,000 characters), document type, target length, audience, focus areas, and whether a decision is needed. Claude produces:

- **Bottom line** — the single most important takeaway, stated first, in 1–2 sentences
- **Summary** — the executive summary body at requested length, paraphrased in Claude's own words
- **Key findings** — standalone bullets of specific findings from the source
- **Numbers that matter** — key figures with context, pulled out as visual stat cards
- **Uncertainties or caveats** — things the source itself flags as unresolved or contested
- **Decision recommendation** — if a decision was requested, a clear recommendation based on source content
- **What was cut** — brief note on what detail was omitted, so the reader knows what to check the source for
- **Read time saved estimate**

HTML output styled as a clean executive brief with a highlighted bottom-line box and stat cards for key numbers.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/condense-to-executive-summary \
  -H "Content-Type: application/json" \
  -d '{
    "document_type": "market_research",
    "target_length": "half_page",
    "audience": "CEO and board, deciding whether to enter the enterprise segment",
    "focus_areas": ["market size", "competitive landscape", "go-to-market recommendation"],
    "decision_needed": "Should we build an enterprise tier in the next 2 quarters?",
    "reply_email": "ceo@flowdesk.com",
    "source_document": "[paste the full research report, spec, or analysis document here — min 200 chars, up to 15,000]"
  }'
```

**Required:** `source_document`, `document_type`

---

## Document types

`research_report`, `technical_spec`, `financial_analysis`, `market_research`, `project_status`, `strategy_doc`, `legal_document`, `meeting_notes`, `other`

Type is used mainly for framing and doesn't change the summarization approach significantly, since the summary is driven primarily by content and target length.

---

## Target lengths

`one_paragraph` (very brief), `half_page` (standard executive summary), `one_page` (more detailed but still condensed)

---

## Never fabricates, never lifts verbatim

Claude is instructed to paraphrase in its own words rather than lifting sentences from the source, and to never add findings, numbers, or claims not present in the source document. This keeps the summary both legally safer (no accidental reproduction of copyrighted text) and more genuinely useful (a real synthesis, not a copy-paste with cuts).

---

## Decision-oriented summaries

Pass `decision_needed` when the summary is meant to support an actual decision. Claude will include a specific `decision_recommendation` based on what the source document supports — not inventing a recommendation beyond what the evidence in the source justifies.

---

## Limitations

- Source document capped at 15,000 characters (~2,500 words). For longer documents, either pass the most decision-relevant sections or summarize in stages.
- The summary quality depends on source clarity — a poorly organized source document produces a summary that reflects that disorganization, though Claude will still extract the substance.

---

## License

MIT.
