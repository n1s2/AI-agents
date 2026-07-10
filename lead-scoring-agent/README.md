# lead-scoring-agent

Treating every inbound lead the same wastes rep time on low-fit contacts and lets hot leads go cold. This scores each lead across three dimensions — fit (ICP match), intent (buying signals), and authority (right person) — researches the company via Tavily, and routes to the right action: immediate outreach, nurture, demo scheduling, disqualification, or partner referral. Hot leads trigger an immediate alert to the CRM owner.

---

## What it does

Takes lead details: company, contact, engagement signals, form responses, your ICP criteria, and disqualifiers. Searches Tavily for company context (size, funding, recent news). Claude produces:

- Lead score 0–100 and grade A/B/C/D
- Component scores: fit (ICP match), intent (buying signals), authority (decision maker vs end user)
- Score rationale (2–3 sentences)
- Specific fit signals and intent signals detected
- Authority assessment
- Disqualifying flags hit (if any)
- Routing recommendation: `hot_immediate`, `demo_ready`, `nurture`, `disqualify`, or `route_to_partner`
- Routing rationale
- Suggested opener (what the rep should say/write to this specific lead)
- Next best action (specific, for the next 24 hours)

For `hot_immediate` or `demo_ready` leads: fires immediate plain-text alert to the CRM owner. For all leads: emails full HTML score report.

---

## Stack

n8n, Tavily API, Anthropic Claude (claude-sonnet-4-20250514), SMTP.

---

## Setup

**Env vars:** `TAVILY_API_KEY`, `FROM_EMAIL`

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/score-lead \
  -H "Content-Type: application/json" \
  -d '{
    "lead_id": "LEAD-2025-0441",
    "company_name": "Beacon Logistics",
    "contact_name": "Tanya Okonkwo",
    "contact_title": "VP Operations",
    "contact_email": "tanya@beaconlog.com",
    "industry": "Logistics",
    "company_size": "80-120 employees",
    "location": "Chicago, US",
    "lead_source": "Demo request form",
    "engagement_signals": [
      "Requested demo via website form",
      "Visited pricing page 3 times in last week",
      "Downloaded ops efficiency guide",
      "Company visited /enterprise page"
    ],
    "form_responses": "We currently use a combination of spreadsheets and Asana but it is getting unwieldy as we grow. Looking for something our ops team can actually use without training. Need to make a decision in the next 30-45 days.",
    "ideal_customer_profile": "Ops-heavy company 50-500 employees, decision maker in operations or COO-level, frustrated with spreadsheets or overcomplicated PM tools, in US/Canada, B2B.",
    "product_fit": "Best for ops-intensive teams managing recurring processes. High fit with logistics, agencies, professional services.",
    "disqualifiers": ["Government/public sector", "Less than 10 employees", "Developer tools company"],
    "crm_owner_email": "jake@flowdesk.com",
    "reply_email": "sales@flowdesk.com"
  }'
```

**Required:** `company_name`, `contact_name`

---

## Routing decisions

| Routing | Meaning |
|---|---|
| `hot_immediate` | Strong ICP fit + active intent + decision maker — rep should reach out within 2 hours |
| `demo_ready` | Good fit and intent — schedule demo within 24 hours |
| `nurture` | Fit but weak intent or authority — add to nurture sequence |
| `disqualify` | Doesn't meet ICP criteria or hit a disqualifier |
| `route_to_partner` | Right problem, wrong fit for direct (competitor, geo, segment mismatch) |

---

## Suggested opener

Not a generic opener — the `suggested_opener` field contains 1–2 sentences that reference something specific about this company or their stated situation. It's a starting point, not a script.

---

## Limitations

- Research is Tavily web results — company size and funding from search may be approximate. For precision, supplement with enrichment tools (Clearbit, Apollo) and pass that data in `company_size` and `annual_revenue`.
- Score calibration depends on ICP quality. Vague ICP produces vague scores. Specific ICP ("ops managers at 50–500 person logistics companies, frustrated with Asana") produces useful differentiation.

---

## License

MIT.
