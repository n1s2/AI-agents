# sales-territory-planner

Territory design done poorly creates coverage gaps, rep conflicts, and unequal workloads that hurt both morale and revenue. A junior rep given a mature enterprise territory fails. An enterprise specialist given a high-velocity SMB territory grinds. This maps your reps' strengths to the right segments, balances workload, identifies gaps, and produces a quota allocation rationale and 90-day activation plan.

---

## What it does

Takes company name, product description, reps (with names, regions, strengths, experience), total market or account base, segmentation criteria, total quota, existing territories, and known challenges. Claude designs:

- Territory strategy (overall approach and rationale)
- Per-rep territory assignments — each with: segments owned, account count estimate, quota share %, why this rep suits this territory, priority accounts to go after first, potential conflicts to watch
- Coverage gaps (segments or regions not clearly owned)
- Quota allocation table with rationale per rep
- Ramp notes for new or transitioning reps
- 90-day activation actions

HTML report shows each territory as a color-coded card per rep, quota table, gaps list, and action plan.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/plan-sales-territory \
  -H "Content-Type: application/json" \
  -d '{
    "company_name": "Flowdesk",
    "product_description": "Project management for small operations teams. ACV $1,500-$25,000.",
    "currency": "USD",
    "total_quota": 2400000,
    "planning_goal": "maximize_coverage",
    "total_accounts_or_market": "~8,000 SMB accounts in our ICP (ops teams at 10-100 person companies) across North America. Current penetration ~5%. Key verticals: logistics, professional services, marketing agencies.",
    "segmentation_criteria": [
      "Company size: SMB (10-50), mid-market (51-200)",
      "Vertical: logistics, professional services, agencies, tech startups",
      "Geography: West Coast, East Coast, Central"
    ],
    "known_challenges": "Sarah is our best closer but struggles with long-cycle enterprise deals. Marcus is new (4 months in) and still ramping. We have no rep covering the Central US.",
    "reply_email": "revops@flowdesk.com",
    "reps": [
      {"name": "Sarah Kim", "region": "West Coast", "strengths": "High-velocity SMB, outbound prospecting, strong logistics vertical network", "current_accounts": 180, "quota": 720000, "years_experience": 5},
      {"name": "James Osei", "region": "East Coast", "strengths": "Mid-market deals, professional services vertical, consultative selling", "current_accounts": 120, "quota": 840000, "years_experience": 8},
      {"name": "Marcus Webb", "region": "National", "strengths": "Tech startup vertical, product-led growth motions, still ramping", "current_accounts": 45, "quota": 480000, "years_experience": 1},
      {"name": "Priya Nair", "region": "East Coast", "strengths": "Agency vertical, outbound, high email response rates", "current_accounts": 95, "quota": 360000, "years_experience": 3}
    ]
  }'
```

**Required:** `company_name`, `reps`, `total_accounts_or_market`

---

## Rep matching logic

Claude reasons explicitly about why each rep suits their territory — not just balanced headcount. A rep with agency vertical experience owns the agency segment even if it's geographically broader. A ramping rep gets a territory with higher inbound volume and simpler deals. The `why_this_match` field per territory explains the assignment.

---

## Coverage gaps

Segments or geographies with no clear owner are flagged explicitly. In the example above, the Central US gap would surface as a coverage gap with a recommendation (expand an existing territory or flag as a next hire).

---

## Limitations

- The plan is qualitative — it doesn't integrate with a CRM to pull actual account lists or model revenue by segment. Use this as a strategic starting point and validate with your actual account data.
- Territory transitions (moving accounts from one rep to another) involve relationship risk that isn't modeled here. The plan identifies the right design; the implementation timeline should be managed carefully.

---

## License

MIT.
