# vendor-comparison-matrix

Vendor comparisons done in a spreadsheet with checkmarks in feature columns don't help you decide — every vendor's sales page claims to check every box. This searches for public comparison data via Tavily, scores each vendor honestly against your specific criteria (not artificially balanced), flags deal-breakers, and ends with an actual recommendation and rationale rather than leaving you to eyeball a table.

---

## What it does

Takes 2–8 vendors (with pricing info, known features, and notes), evaluation criteria, weightings, deal-breakers, and budget constraint. Searches Tavily for public comparison data. Claude produces:

- **Comparison summary** — overall picture, standout vendor if any, key tradeoffs
- **Matrix** — each vendor scored 1–10 on each criterion, with specific rationale and confidence level (high/medium/low) per score, weighted total, and deal-breaker flag
- **Strengths and weaknesses by vendor**
- **Best fit scenarios** — for each vendor, the specific situation where it's the right choice
- **Recommendation** — recommended vendor, rationale, confidence, and caveats that could change the recommendation
- **Questions to ask vendors** — specific things to verify in follow-up calls
- **Information gaps** — where public information was insufficient for full confidence

HTML matrix table with color-coded scores, vendor profile cards, and recommendation highlighted with confidence level.

---

## Stack

n8n, Tavily API, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/compare-vendors \
  -H "Content-Type: application/json" \
  -d '{
    "decision_context": "Choosing a customer support helpdesk tool for a 45-person B2B SaaS company. Currently on shared inbox, need ticketing, SLA tracking, and a knowledge base.",
    "budget_constraint": "Under $15/agent/month, team of 6 support agents",
    "reply_email": "cs-lead@flowdesk.com",
    "evaluation_criteria": [
      "Ticketing and SLA management",
      "Knowledge base / help center",
      "Integration with Slack",
      "Reporting and analytics",
      "Ease of setup",
      "Price at our team size"
    ],
    "weightings": {"Ticketing and SLA management": 3, "Price at our team size": 2},
    "deal_breakers": ["No API access", "No SOC 2 compliance"],
    "vendors": [
      {"name": "Zendesk", "pricing_info": "$19-115/agent/month depending on tier", "known_features": "Full ticketing, SLA, knowledge base, extensive integrations, robust reporting", "notes": "Industry standard but can be complex to set up"},
      {"name": "Help Scout", "pricing_info": "$20-65/agent/month", "known_features": "Shared inbox style, simpler than Zendesk, has knowledge base and light reporting", "notes": "Known for simplicity, less robust SLA management"},
      {"name": "Freshdesk", "pricing_info": "Free tier, then $15-79/agent/month", "known_features": "Ticketing, SLA, knowledge base, Slack integration, has a free tier", "notes": "Good price point, mixed reviews on reporting depth"}
    ]
  }'
```

**Required:** `vendors`, `evaluation_criteria`

---

## Honest scoring

Claude is instructed to score based on evidence, not to artificially balance scores across vendors to seem "fair." If one vendor genuinely outperforms on most criteria, the matrix reflects that. Confidence level per score signals when public information limits certainty.

---

## Deal-breakers

Pass hard requirements as `deal_breakers`. Any vendor that fails a deal-breaker gets explicitly flagged in the matrix header and profile card, even if their other scores are strong. This prevents a vendor with a good weighted total from being recommended when they fail a non-negotiable requirement.

---

## Weightings

Pass a `weightings` object to weight specific criteria more heavily in the total score. Criteria not in the weightings object default to weight 1.

---

## Limitations

- Public research via Tavily reflects publicly available comparison content, not hands-on evaluation. For final decisions, always do live demos and reference checks — the `questions_to_ask_vendors` field is designed to guide those calls.
- Up to 8 vendors per call, sufficient for most shortlisted comparisons.

---

## License

MIT.
