# sales-call-debrief-agent

Most sales call notes are a summary of what was said, not what matters. A debrief that helps a rep actually close a deal needs to extract: what the prospect actually cares about (in their own words), the real objections (not just the stated ones), where you stand relative to competitors, who actually makes the buying decision, and the specific next steps that will move the deal forward — not "follow up next week."

This takes raw call notes and produces a structured debrief with a deal health score, buying signals, objections with rebuttals, pain points in the prospect's language, decision process mapping, competitive positioning, critical next steps, and a coaching observation for the manager. Emails the rep, manager, or both.

---

## What it does

1. Accepts: prospect company, contact, rep name, deal stage, call notes, deal value, competitors, product
2. Claude extracts and structures:
   - Deal summary (2–3 sentences on where the deal stands)
   - Buying signals (specific things said that indicate genuine interest)
   - Objections with type classification and suggested rebuttal
   - Pain points in the prospect's language
   - Decision process: who signs, other stakeholders, timeline, evaluation criteria
   - Competitive situation: who else they're evaluating, your position
   - Deal health score (1–10) with rationale and biggest risk
   - Next steps with owner (rep/prospect/manager/SE) and due date, critical items flagged
   - Coaching note for the manager
3. Builds color-coded HTML debrief (score shown prominently, critical next steps highlighted)
4. Emails rep, manager, and/or reply email
5. Returns full JSON

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP.

---

## Setup

**Env vars:** `FROM_EMAIL`
**Credentials:** Anthropic API (LangChain node), SMTP.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/sales-debrief \
  -H "Content-Type: application/json" \
  -d '{
    "prospect_company": "Meridian Logistics",
    "prospect_contact": "Tanya Okonkwo",
    "prospect_title": "VP Operations",
    "rep_name": "Jake Reyes",
    "rep_email": "jake@company.com",
    "deal_stage": "demo",
    "call_date": "2025-05-21",
    "deal_value": 48000,
    "currency": "USD",
    "product_offered": "Flowtrack warehouse operations platform",
    "competitors_involved": ["SAP Extended Warehouse", "Oracle WMS"],
    "manager_email": "sales-manager@company.com",
    "call_notes": "30-min demo with Tanya and her ops lead Marcus. Tanya was engaged throughout especially when we showed the mobile offline mode — she said their current system requires wifi and their rural warehouse has dead zones which is a real problem. Marcus was skeptical about the API integrations asking multiple times about their legacy ERP (Sage 200). Tanya said they need to make a decision by end of Q2 because their contract with current vendor expires July 1. Budget is approved in principle but she mentioned her CFO wants to see an ROI calculation. They are also looking at SAP but she said it feels like overkill for their size. Oracle was mentioned but she seemed dismissive. We did not get to show the analytics module — ran out of time. Next call should include Marcus and their IT lead. Jake pushed for a next meeting at the end but Tanya said she needs to check her calendar."
  }'
```

**Required:** `prospect_company`, `call_notes`, `rep_name`

---

## Deal stages

`discovery`, `demo`, `technical_evaluation`, `proposal`, `negotiation`, `closing`, `renewal`

---

## Deal health score

1–10 based on: strength of pain points, buying signals, decision maker access, timeline clarity, competitive position, and identified risks. Claude explains the rationale and names the single biggest risk to the deal.

Score colors: 7–10 green, 5–6 amber, 1–4 red.

---

## Coaching note

The `coaching_note` field is a single specific observation for the manager — not generic encouragement. Based on the notes, it might flag a missed discovery question, a strong handling of a competitor objection, a failure to get a committed next step, or a rapport-building skill used well. This stays off the email to the rep if you prefer by routing selectively.

---

## Limitations

- Analysis quality depends on note quality. Two-sentence notes produce thin debriefs. Detailed notes produce specific, useful output.
- This is a standalone call analysis — it doesn't maintain deal history across calls. For longitudinal deal tracking, log outputs to a CRM or Sheet and include prior context in new call notes.

---

## Ideas

- [ ] CRM integration: push deal health score and next steps to Salesforce/HubSpot after each call
- [ ] Win/loss pattern: track deal health scores over time and correlate with outcomes
- [ ] Competitor playbooks: when a competitor is detected, pull the relevant battle card and include it in the debrief

---

## License

MIT.
