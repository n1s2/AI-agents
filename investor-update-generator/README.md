# investor-update-generator

Investor updates that only share good news erode trust. Updates that are walls of text don't get read. Updates without a clear ask waste the relationship. This generates a concise, honest monthly or quarterly investor update: leading with metrics, sharing wins and challenges with equal candor, and closing with a specific ask so investors know exactly how to help.

---

## What it does

Takes company name, period, key metrics, highlights, challenges, product/sales/team updates, fundraising status, and ask of investors. Claude writes:

- **Email subject line**
- **Opening** — the most important thing that happened this period, in plain direct language
- **Metrics summary** — what moved, direction, and brief context on why
- **Highlights** — 2–5 specific wins (concrete, not generic)
- **Challenges** — honest description of each challenge with what the team is doing about it
- **Product, sales/growth, team** — narrative sections
- **Fundraising status** — if applicable
- **Ask of investors** — each ask with context on what the investor can specifically do
- **Closing** — forward-looking, confident without being spin

Also generates: a 100-word TL;DR version for investors who prefer brief, and a private founder notes field for anything better shared one-on-one rather than in the group update.

Can auto-send to the investor list immediately, or route to reply email for review first.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-investor-update \
  -H "Content-Type: application/json" \
  -d '{
    "company_name": "Flowdesk",
    "period": "May 2025",
    "frequency": "monthly",
    "stage": "Series A / $52k MRR",
    "recipient_emails": "investors@flowdesk.com",
    "reply_email": "jake@flowdesk.com",
    "metrics": {
      "MRR": "$54,200 (+4.2% MoM)",
      "ARR": "$650k",
      "New customers": "18",
      "Churned customers": "3",
      "Net MRR expansion": "+$2,100",
      "Runway": "19 months"
    },
    "highlights": [
      "Shipped Notion bidirectional sync — first integration partnership. 23 customers connected in first week.",
      "Signed Beacon Logistics (ACV $14,400) — our largest deal to date",
      "Hired Sara Kim as second backend engineer, starts June 9"
    ],
    "challenges": [
      "Conversion rate dropped from 4.2% to 3.6% in May — we think it's the checkout flow change we shipped on May 12. Reverting and re-testing.",
      "Lost 3 customers this month vs 1 last month — all three cited wanting Asana because their dev team uses it. Working on a response to this objection."
    ],
    "product_updates": "Shipped Notion sync (FD-1201), bulk task assignment (FD-1195), and resolved the Safari iOS logout bug. June focus: email digest feature and onboarding flow improvements.",
    "sales_updates": "Pipeline at $120k ARR. 3 enterprise conversations in flight (Meridian, PacificFreight, WestCoast Agency). Hired first SDR who starts June 23.",
    "team_updates": "Now 9 people. Sara Kim (backend) joining June 9. Actively hiring for head of CS.",
    "fundraising_status": "Not currently raising. Will revisit Series A conversation in Q4 when we hit $1M ARR.",
    "ask_of_investors": [
      "Intros to ops leaders at logistics or agency companies (10-100 employees) — our best ICP right now",
      "Anyone who has built a CS function from scratch at a similar stage — looking for advisor or potential hire"
    ],
    "founder_notes": "The conversion rate drop is more concerning than I let on in the update. We are digging into session recordings this week. Will reach out individually if it looks systemic."
  }'
```

**Required:** `company_name`, `period`

---

## Honest about challenges

Claude is instructed to write challenges with the same directness as highlights — honest about what is hard and specific about what the team is doing in response. Investors who read vague updates ("the market is challenging") are less able to help than investors who read specific ones ("we lost 3 customers this month, all to Asana, and here's our response").

---

## Private founder notes

The `founder_notes` field captures things better shared one-on-one. The agent places these in a separate section at the bottom of the HTML doc, clearly marked for personal sharing rather than mass send.

---

## Sending options

- Pass `recipient_emails` (comma-separated) to send immediately to the investor list
- Pass `reply_email` to receive the HTML preview for review first
- Pass both to get the preview and still send to the list

For review-before-sending, use `reply_email` only, review the draft, then trigger a separate send when ready.

---

## TL;DR version

The `one_paragraph_version` field is a 100-word standalone summary for investors who prefer a brief. Some investors read only this; some read the full update. Having both means everyone gets the information in the format they prefer.

---

## Limitations

- Update quality depends on input specificity. Vague inputs ("product is going well") produce vague updates. Specific inputs ("shipped Notion sync, 23 customers connected in week 1") produce specific, credible updates.
- The agent writes the update from what you provide — it doesn't pull from your CRM, billing system, or analytics platform directly. Pass the metrics you want included.

---

## License

MIT.
