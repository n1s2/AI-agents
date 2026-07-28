# product-feedback-prioritizer

A backlog of 200 feature requests sorted by vote count isn't a prioritized roadmap. The most-voted item might be a low-ICP segment asking for something that doesn't fit your strategy. The item with two votes from your highest-revenue accounts might be the thing that moves ARR. This groups feedback into themes, surfaces the underlying need behind each request, weights by revenue and ICP fit, and produces a P0–P3 tier breakdown with build/decline recommendations and a "what not to build" list.

---

## What it does

Takes up to 60 feedback items (each with source, customer segment, plan tier, revenue, vote count, and text). Also takes product strategy, current roadmap, and ICP description for alignment scoring. Claude produces:

- **Prioritization summary** — what patterns stand out, what the feedback collectively reveals about the product
- **Themes** — grouped clusters, each with: underlying need (not the surface request), ICP alignment, strategy alignment, revenue-weighted signal, priority tier (P0/P1/P2/P3), build recommendation (build/investigate_further/defer/decline), and rationale
- **Top individual items** — priority score, action, and specific reason why this item was prioritized
- **Items to decline** — with specific reasons
- **Product gaps revealed** — structural gaps the feedback collectively points to
- **What not to build** — categories that sound reasonable but should be declined
- **Quick wins** — items flagged for fast implementation
- **Data to gather** — what additional information would improve confidence

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/prioritize-feedback \
  -H "Content-Type: application/json" \
  -d '{
    "product_name": "Flowdesk",
    "product_strategy": "Win ops-heavy SMBs (15-100 employees) who are outgrowing spreadsheets. Grow through product-led motion — easy onboarding, no IT required. Do not chase enterprise for next 12 months.",
    "current_roadmap": "Next quarter: email digest, onboarding improvements, Slack integration. Not planning: mobile app, advanced reporting, SSO.",
    "icp_description": "Operations managers at logistics, agency, or professional services companies, 15-100 employees, replacing spreadsheets.",
    "reply_email": "product@flowdesk.com",
    "feedback_items": [
      {"id": "FB-001", "source": "nps_survey", "customer_segment": "logistics", "plan_tier": "business", "revenue": 2400, "votes": 1, "type": "feature_request", "text": "Need to assign tasks to multiple people at once. I have to click into each task individually which takes forever when rebalancing after someone leaves."},
      {"id": "FB-002", "source": "intercom", "customer_segment": "agency", "plan_tier": "starter", "revenue": 600, "votes": 3, "type": "feature_request", "text": "Would love a Gantt chart view to show project timelines to clients"},
      {"id": "FB-003", "source": "churn_exit", "customer_segment": "tech_startup", "plan_tier": "business", "revenue": 1800, "votes": 1, "type": "complaint", "text": "Left because our dev team uses Asana and we needed everything in one place. No hard feelings, just needed the integration."},
      {"id": "FB-004", "source": "in_app_survey", "customer_segment": "logistics", "plan_tier": "business", "revenue": 3600, "votes": 5, "type": "feature_request", "text": "Daily email summary of what my team completed and what is blocked. I check the app but sometimes miss things when traveling."},
      {"id": "FB-005", "source": "sales_call", "customer_segment": "professional_services", "plan_tier": "business", "revenue": 4800, "votes": 1, "type": "feature_request", "text": "We need SSO — our IT policy requires it for any tool with more than 10 users. Would sign a 2 year deal if you had it."}
    ]
  }'
```

**Required:** `feedback_items`

---

## Revenue weighting

Claude considers revenue alongside vote count. A single request from a $4,800 ARR customer with high ICP fit outweighs 5 votes from $600 starter accounts in segments outside your ICP. Pass `revenue` per item (annual or MRR, just be consistent) for weighted signal.

---

## Underlying needs vs feature asks

Every theme surfaces the underlying need behind the requests — not "they want email digest" but "ops managers need visibility into team status without opening the product every day." This reframing often reveals that multiple requests point to the same underlying need, or that a simpler solution exists than the feature requested.

---

## What not to build

Claude explicitly lists categories of requests that sound reasonable but should be declined — with rationale. For the Flowdesk example: "Gantt charts: requested by agency segment (outside ICP) and implies project management mental model we don't serve. Building this would add complexity without converting target customers." This is as important as the build list.

---

## Limitations

- Prioritization is relative to the strategy and ICP you provide. Vague strategy input ("build a great product") produces generic prioritization. Specific strategy input ("win ops-heavy SMBs, don't chase enterprise") produces opinionated, useful prioritization.
- 60 items maximum per call. For larger backlogs, run the agent on themed subsets (e.g., all integrations requests separately from all UI requests) and synthesize the outputs.

---

## License

MIT.
