# customer-interview-analyzer

Interview transcripts pile up faster than teams can synthesize them. Reading back through 45 minutes of conversation to extract the two insights that matter takes as long as the interview itself. This takes a raw transcript and produces a structured analysis: the job to be done in the customer's language, pain points with severity and pull quotes, hypothesis verdicts, surprising findings, and specific product implications.

---

## What it does

Takes an interview transcript (up to 10,000 characters), interviewee role, research goal, product area, existing hypotheses to test, and prior findings summary. Claude produces:

- **Interview summary** — who this person is, what they're trying to accomplish, the most important thing the interview revealed
- **Top job to be done** — the primary outcome they're trying to achieve, in their language
- **Current workflow** — how they accomplish this today, including tools, steps, and workarounds
- **Pain points** — each with severity (critical/significant/minor), a pull quote, and the workaround they use
- **Desired outcomes** — what they want to be true (outcomes, not features)
- **Surprising findings** — things that contradicted assumptions or were unexpected
- **Hypothesis assessment** — each hypothesis tested with a verdict (confirmed/contradicted/partially_confirmed/not_addressed) and the evidence
- **Pull quotes** — the most shareable verbatim quotes, with context and suggested use (persona doc, sales deck, design review)
- **Follow-up questions** — what this interview raised that future interviews should probe
- **Product implications** — specific things the product team should consider

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Setup

**Env vars:** `FROM_EMAIL`
**Credentials:** Anthropic API (LangChain node), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/analyze-customer-interview \
  -H "Content-Type: application/json" \
  -d '{
    "interview_id": "INT-2025-014",
    "interviewee_role": "Operations Manager",
    "interviewee_name": "Rachel Park",
    "interviewee_company": "Meridian Logistics",
    "interview_date": "2025-05-30",
    "product_area": "Task assignment and workload management",
    "research_goal": "Understand how ops managers handle workload rebalancing when team composition changes",
    "existing_hypotheses": [
      "Ops managers spend significant time reassigning tasks manually when team members leave or join",
      "The pain is worse at companies with more than 20 tasks per person",
      "Excel is the primary alternative tool they turn to for bulk operations"
    ],
    "previous_findings_summary": "Previous 3 interviews confirmed manual reassignment is painful. Two interviewees mentioned needing to do this at least monthly. One mentioned using a Zapier workaround.",
    "reply_email": "research@flowdesk.com",
    "interview_transcript": "[Paste the full interview transcript here — minimum 100 characters, up to 10,000]"
  }'
```

**Required:** `interview_transcript`, `interviewee_role`

---

## Hypothesis testing

Pass your existing hypotheses in `existing_hypotheses`. Each gets assessed against the transcript with a verdict and specific evidence. This builds pattern recognition across interviews — after 5 interviews, you can see which hypotheses are consistently confirmed, which are getting contradicted, and which aren't being addressed yet.

---

## Pull quotes

The `pull_quotes` field extracts the most usable verbatim quotes from the transcript with context and suggested use cases. These are interview-quality quotes for personas, sales decks, design reviews, or stakeholder presentations — not paraphrases.

---

## Surprising findings

Claude is specifically instructed to flag things that contradicted assumptions or were unexpected — not just confirm what the interviewer expected to hear. This is where the most valuable research signal often lives.

---

## Limitations

- Input is capped at 10,000 characters (~1,500 words). For longer transcripts, pass the full transcript (it'll be truncated at the end) or the most substantive portion.
- Pull quotes are verbatim where possible from the transcript, but if the transcript is edited or paraphrased, the quotes will reflect that.
- This analyzes one interview at a time. For cross-interview synthesis, run this on each interview, then use a separate agent (like user-feedback-synthesizer) to aggregate findings across sessions.

---

## License

MIT.
