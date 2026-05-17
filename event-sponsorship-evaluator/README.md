# event-sponsorship-evaluator

Sponsorship requests land in marketing inboxes constantly. Most are declined by default or accepted based on gut feel. Neither is a strategy.

The actual question — does this event reach our target audience, is the package worth the price, and what should we negotiate — takes 30 minutes of research and a clear framework to answer properly. This does it in 30 seconds.

You submit the event details, the sponsorship package, the asking price, and your company profile. It searches the web for context on the event, then Claude evaluates the opportunity like a marketing strategist: audience alignment, cost per attendee, package quality, competitor activity, strategic positioning, risks, and specific negotiation recommendations including a counter-offer suggestion.

---

## What it does

1. Accepts a POST: event name, type, date, attendees, audience description, sponsorship package, asking price, company profile, target audience, previous sponsorships, competitors sponsoring
2. Searches Tavily for current event information
3. Claude evaluates across six dimensions: audience fit, price fairness (with cost-per-attendee), package quality (strong vs weak elements), strategic considerations, risks, and negotiation recommendations
4. Returns a verdict: `strong_yes`, `yes_with_conditions`, `maybe`, `lean_no`, or `hard_no` with a 0-100 score
5. Lists specific questions to ask the organizer before signing
6. Builds a formatted HTML report
7. Emails if `reply_email` provided

---

## Stack

- **n8n** — webhook + workflow
- **Tavily API** — event research
- **Anthropic Claude** (claude-sonnet-4-20250514) — evaluation
- **SMTP** — optional email delivery

---

## Setup

### 1. Environment variables

```
TAVILY_API_KEY=tvly-your-key
FROM_EMAIL=marketing@yourcompany.com
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** (optional)

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/evaluate-sponsorship \
  -H "Content-Type: application/json" \
  -d '{
    "event_name": "SaaStr Annual 2025",
    "event_type": "conference",
    "event_date": "2025-09-10",
    "event_location": "San Francisco, CA",
    "expected_attendees": 12000,
    "audience_description": "B2B SaaS founders, VCs, and operators. Series A-C companies.",
    "sponsorship_package": "Gold Sponsor",
    "package_description": "10x10 booth, logo on app and website, 2 breakout speaking slots, 15 passes, post-event email to 4k opt-in attendees",
    "asking_price": 45000,
    "currency": "USD",
    "company_profile": "Revenue operations software for B2B SaaS companies 50-500 employees. ICP: VP Sales and RevOps at Series A-C. ACV $24k.",
    "target_audience": "VP Sales, RevOps leaders, CROs at Series A-C SaaS",
    "marketing_budget": 400000,
    "previous_sponsorships": "SaaStr Europa 2024: 38 qualified leads, 4 closed deals at $96k ARR",
    "competitors_sponsor": "Clari is Platinum, Gong is Gold",
    "reply_email": "cmo@company.com"
  }'
```

**Required:** `event_name`, `sponsorship_package`, `asking_price`, `company_profile`

---

## Verdict levels

| Verdict | Meaning |
|---|---|
| `strong_yes` | Clear fit, fair price, move fast |
| `yes_with_conditions` | Good opportunity but negotiate specific terms first |
| `maybe` | Borderline — depends on clarifications |
| `lean_no` | Probably not worth it |
| `hard_no` | Clear misalignment or poor value |

---

## The negotiation section

Every evaluation includes specific negotiation asks with rationale — not generic "try to get a discount" but concrete requests based on the package analysis. The counter-offer suggestion gives a specific alternative price or modification that would make the deal more attractive.

---

## The `company_profile` field

Most important field for accuracy. Include: what you sell, who buys it (title, company size, stage), deal size, and positioning. The more specific, the better Claude can assess audience alignment.

---

## Previous sponsorships

Fill in past results (leads generated, deals closed) and Claude uses it as a benchmark to compare this event against your actual historical ROI.

---

## Without Tavily

Remove **Research Event** and **Merge Context** nodes, connect **Valid?** directly to **Claude Sponsorship Evaluator**, replace `{{ $json.eventContext }}` with a static string. Claude uses its own knowledge of well-known events.

---

## Limitations

- Web research quality varies by event size. Smaller events may have little indexed content.
- Attendance and list size claims from organizers are unverified — flag anything that seems inflated.
- Strategic assessment only, not financial modeling with your specific conversion rates.

---

## Ideas

- [ ] Sponsorship history tracker: log evaluations and outcomes, build a benchmark database
- [ ] Budget allocation: submit multiple events, Claude recommends how to split a fixed budget
- [ ] Post-event ROI logger: compare actual results to the pre-event score
- [ ] Counter-offer email drafter: write the negotiation email from the evaluation output

---

## License

MIT.
