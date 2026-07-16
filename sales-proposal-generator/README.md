# sales-proposal-generator

Generic proposals lose deals. The prospect reads it and thinks "this could have been sent to anyone." A good proposal opens with their specific situation, connects the solution to the exact problems discussed on calls, makes pricing feel like the natural result of the value being delivered, and ends with clear next steps. This generates a complete, personalized proposal from call notes and solution details.

---

## What it does

Takes prospect company, product/service, solution description, problems addressed, pricing, case studies, and call notes. Claude writes:

- **Email cover note** — short personal note to accompany the proposal, references something specific
- **Executive summary** — 3–4 sentences for executives who weren't on discovery calls
- **The challenge** — the prospect's specific situation in their own language (pulled from call notes)
- **Our approach** — solution narrative connecting directly to the named problems
- **Why us** — specific reasons this company suits this prospect (not generic strengths)
- **Investment** — package, pricing structure, ROI framing
- **Implementation** — timeline overview, what the vendor handles vs what the prospect provides
- **Customer evidence** — case studies formatted as social proof
- **Next steps** — table with step, owner (prospect/vendor), and timing
- **Terms summary**

HTML formatted as a clean professional proposal document with serif type.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-proposal \
  -H "Content-Type: application/json" \
  -d '{
    "prospect_company": "Beacon Logistics",
    "prospect_contact": "Tanya Okonkwo",
    "prospect_title": "VP Operations",
    "product_name": "Flowdesk",
    "company_name": "Flowdesk Inc.",
    "rep_name": "Jake Reyes",
    "valid_until": "2025-06-30",
    "contract_term": "12 months",
    "solution_description": "Flowdesk Operations Hub for Beacon Logistics — includes unlimited users, Slack and email integrations, weekly check-in automation, and dedicated onboarding support.",
    "proposed_package": "Operations Hub — Annual",
    "pricing": "USD 18,000/year (USD 1,500/month)",
    "pricing_currency": "USD",
    "implementation_timeline": "Week 1: setup and admin training. Week 2: team rollout. Week 3: Slack and email integration. Week 4: first automated weekly review.",
    "problems_addressed": [
      "3-hour weekly planning meetings consuming ops team time",
      "No central visibility into task ownership across 4 warehouse locations",
      "Manual handoffs causing delays when team members are unavailable"
    ],
    "key_value_props": [
      "Replaces 3-hour weekly meeting with a 20-minute async check-in",
      "Single view of all tasks across locations for ops lead",
      "Automatic escalation when tasks go unacknowledged"
    ],
    "case_studies": [
      "Meridian Transport: reduced planning overhead by 40% in first 90 days. 3-hour weekly ops meeting eliminated.",
      "PacificFreight: rolled out to 120 users across 6 sites in 3 weeks with zero IT involvement."
    ],
    "call_notes": "Tanya mentioned her biggest frustration is that she spends her Sunday evenings preparing for the Monday ops meeting. She said quote this is not what I was hired to do. Her team is spread across Chicago, Dallas, Memphis, and LA. Marcus (ops lead in Dallas) currently maintains a shared Excel file that nobody trusts. They need a decision by June 15 due to a board presentation on ops efficiency.",
    "reply_email": "jake@flowdesk.com"
  }'
```

**Required:** `prospect_company`, `product_name`, `solution_description`

---

## Call notes make the difference

The `call_notes` field is what makes this feel personal. Claude pulls the prospect's language directly into the problem statement and solution narrative. Tanya's "Sunday evening prep" becomes a specific line in the proposal. The more specific the call notes, the more the proposal reads like it was written for this exact company.

---

## Email cover note

Generated alongside the proposal — a 3–4 sentence personal note referencing something specific from the relationship. Paste this as the email body when sending the proposal document.

---

## Limitations

- The proposal is a draft — verify pricing, package details, and any specific claims before sending.
- Without call notes, the problem statement is derived from `problems_addressed` only — it will be accurate but less personalized.

---

## License

MIT.
