# lease-renewal-negotiator

Most tenants accept whatever rent increase their landlord proposes because they don't know the market, don't know what to say, or assume negotiating will damage the relationship. All three assumptions are usually wrong.

Landlords price renewal increases to what they think they can get, not what the market requires. Long-term tenants have real leverage — vacancy costs landlords a month or more of rent plus repairs and advertising. And a professional, evidence-based counter-offer almost never damages a relationship; it signals you're serious about staying.

This takes your lease situation — current rent, proposed increase, property issues, your history as a tenant — searches the local rental market, and builds a complete negotiation playbook: a specific counter-offer number, a full email draft ready to send, how to respond to likely pushback, how to use property maintenance issues as leverage, and what concessions to negotiate if rent is truly stuck.

Works for residential, commercial, retail, and office leases.

---

## What it does

1. Accepts a POST: property address, current rent, proposed new rent, lease type, years at property, property issues, tenant strengths, market data
2. Searches Tavily for current local rental market rates and typical increase ranges
3. Claude builds a complete negotiation package:
   - Increase assessment: reasonable / above market / significantly above market
   - Market context and typical range for this area
   - Specific counter-offer with fallback and walk-away point
   - Leverage points and weaknesses analysis
   - Full email draft with subject line — ready to send
   - 3 likely landlord pushbacks with word-for-word responses
   - How to use property maintenance issues as leverage
   - Long-term tenant talking points
   - Non-rent concessions to negotiate if base is stuck
   - Timing advice
   - The one thing not to do
4. Builds HTML coaching report
5. Emails if `reply_email` provided
6. Returns full JSON

---

## Stack

- **n8n** — webhook + workflow
- **Tavily API** — rental market research
- **Anthropic Claude** (claude-sonnet-4-20250514) — negotiation coaching
- **SMTP** — optional email delivery

---

## Setup

### 1. Environment variables

```
TAVILY_API_KEY=tvly-your-key
FROM_EMAIL=coach@yourdomain.com
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** (optional)

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/lease-negotiation \
  -H "Content-Type: application/json" \
  -d '{
    "property_address": "12B Ashford Road",
    "property_type": "2-bedroom apartment",
    "city": "Chicago, IL",
    "lease_type": "residential",
    "current_rent": 1850,
    "proposed_new_rent": 2100,
    "currency": "USD",
    "rent_frequency": "monthly",
    "tenant_name": "Jordan Kim",
    "years_at_property": 4,
    "current_lease_term": "12 months",
    "lease_start_date": "2021-06-01",
    "renewal_deadline": "2025-04-30",
    "target_rent": 1950,
    "property_issues": "Bathroom fan broken since October, reported twice with no fix. Hallway lighting flickering for 3 months. Building intercom has been out for 6 weeks.",
    "tenant_strengths": "Never missed or late on a payment in 4 years. Worked from home so no excessive wear. No complaints from neighbors. Have referred two friends who now also rent in the building.",
    "landlord_context": "Small private landlord, owns 3 units in the building. Seems to raise rent every renewal.",
    "negotiation_style": "collaborative",
    "reply_email": "jordan@email.com"
  }'
```

**Required:** `property_address`, `current_rent`, `currency`, `proposed_new_rent`, `lease_type`

---

## Lease types

`residential`, `commercial`, `retail`, `office`, `industrial`, `short_term`

The type shapes the coaching significantly — commercial lease negotiations have different dynamics and available concessions than residential ones.

---

## The counter-offer number

Claude gives a specific number, not a range. Based on market research, the increase percentage, your leverage points, and stated target. The fallback position is the number to accept if the landlord won't meet your counter. The walk-away point clarifies when moving out is genuinely better than renewing.

---

## The email draft

The output includes a complete, ready-to-send email that:
- Opens professionally and expresses intent to renew
- References the proposed increase by percentage
- Makes a specific counter-offer with supporting rationale
- Mentions relevant leverage (years as tenant, payment record, market context)
- Is polite but firm — doesn't apologize for asking

Read it, adjust the tone to sound like you, then send it. The structure is what matters.

---

## Property issues as leverage

This is the most underused tool in lease negotiations. If your bathroom fan has been broken for three months and you've reported it twice, that's not just an inconvenience — it's leverage. A landlord who wants a 14% rent increase while failing to maintain the property is on weak ground.

Fill in the `property_issues` field with anything that hasn't been fixed. Claude incorporates this directly into the negotiation strategy and email, framing it professionally rather than as a complaint.

---

## Long-term tenant argument

The economics of tenant turnover are clear: a landlord who loses a reliable 4-year tenant faces one month or more of vacancy, cleaning, repairs, and advertising costs — often $3,000–8,000+ depending on the unit. That typically exceeds the annual value of a contested rent increase.

Claude writes this as a concrete talking point rather than a vague appeal to loyalty.

---

## Beyond-rent concessions

When landlords genuinely can't move on base rent, there are often other ways to get value:
- Locking in the current rate for 18–24 months instead of 12
- Repairs completed before signing
- Parking included or reduced
- Reduced security deposit on renewal
- One-month rent concession as a signing incentive

Each comes with a specific ask so you know exactly what to say.

---

## Without Tavily

Remove the **Research Rental Market** and **Merge Market Data** nodes, connect **Valid?** directly to **Claude Lease Coach**, replace `{{ $json.marketContext }}` with an empty string. Supply your own market data in the `market_data` field — paste Zillow, Apartments.com, or local rental listing data directly.

---

## Limitations

- Market data from Tavily is web-indexed and may lag current conditions. Supplement with active rental listings from your specific neighborhood for the most accurate comparison.
- Rent increase limits vary by jurisdiction — some cities have rent control or stabilization ordinances that cap increases. Claude gives strategic advice but doesn't provide legal guidance on applicable regulations. Check local tenant protection laws.
- Commercial leases have more variables than residential — CAM charges, tenant improvement allowances, HVAC responsibilities, exclusivity clauses. The coaching covers the negotiation framework but you should review the full lease terms with a real estate attorney for significant commercial deals.

---

## Ideas

- [ ] Rent control checker: given city and lease type, search for applicable rent control ordinances
- [ ] Lease comparison: compare two renewal offers or two properties side by side
- [ ] Move-out cost calculator: given new rent and moving costs, calculate the true cost of leaving vs accepting
- [ ] Renewal tracker: log all your lease renewals over time, track rent increase history

---

## License

MIT.
