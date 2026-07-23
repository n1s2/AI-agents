# competitive-battle-card-writer

Battle cards that list features side-by-side don't help reps win deals. Reps need to know which deal situations favor them, what to say when a prospect raises a competitor objection, which questions to ask to surface competitor weaknesses naturally, and when to walk away. This searches for current competitor information via Tavily and generates a complete, honest battle card including where the competitor actually wins.

---

## What it does

Takes your product, competitor name, descriptions, known strengths and weaknesses, common objections, pricing context, and win/loss notes. Searches Tavily for current competitor reviews and pricing. Claude produces:

- **Competitor overview** — honest 2–3 sentence assessment of who buys them and why
- **Where we win** — deal scenarios with specific talk tracks
- **Where they win** — honest scenarios with reframe/mitigation strategies
- **Feature comparison table** — dimension-by-dimension with advantage flagged
- **Objection responses** — what the prospect says, what NOT to say, what to say instead
- **Discovery landmines** — questions to ask that naturally surface competitor weaknesses
- **Traps to avoid** — things reps commonly say that backfire in this competitive situation
- **Displacement play** — best angle if prospect is already using the competitor
- **Qualify in / walk away signals** — deal signals that favor you vs. not

HTML with green "where we win" and amber "where they win" side-by-side, feature comparison table, and objection cards with avoid/say contrast.

---

## Stack

n8n, Tavily API, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/write-battle-card \
  -H "Content-Type: application/json" \
  -d '{
    "your_product_name": "Flowdesk",
    "your_product_description": "Lightweight project management for ops-heavy teams at small companies. Quick setup, no IT required, built for operators not developers.",
    "competitor_name": "Asana",
    "competitor_description": "Enterprise project management platform. Feature-rich, complex, strong in tech and marketing teams.",
    "target_persona": "Operations manager at 20-100 person company, not technical, frustrated with spreadsheets",
    "pricing_context": "Flowdesk: $8-15/user/month. Asana: $10.99-24.99/user/month plus enterprise pricing. We are competitive on price for small teams.",
    "your_strengths": [
      "Faster to set up — live in under an hour vs days of configuration",
      "Better for non-technical ops teams — no training required",
      "Simpler pricing — no seat minimums or enterprise contracts for SMB"
    ],
    "known_weaknesses": [
      "Asana has stronger integrations marketplace",
      "Better reporting and dashboards for large teams",
      "More established brand — prospects default to it"
    ],
    "common_objections": [
      "We already evaluated Asana and almost went with them",
      "Our dev team uses Asana, we want everything in one place",
      "Asana has way more integrations"
    ],
    "win_loss_context": "We win when the buyer is an ops manager not IT. We lose when there is an existing Asana contract at the company or when IT is the decision maker. Lost 3 deals last quarter where dev team already used Asana.",
    "reply_email": "sales@flowdesk.com"
  }'
```

**Required:** `competitor_name`, `your_product_name`

---

## Honest about where they win

Most battle cards only list competitor weaknesses. This one explicitly documents where the competitor wins and why — because reps need to know what they're actually up against, and prospects will raise these points anyway. Each "where they win" scenario includes a specific reframe or mitigation strategy.

---

## Landmines vs talking points

The `landmines` field is different from talking points — these are questions the rep asks in discovery that naturally lead the prospect to articulate the competitor's weaknesses themselves. "How long did setup take for your last PM tool?" is more effective than "Asana takes months to implement."

---

## Traps to avoid

Claude includes `traps_to_avoid` — specific things reps commonly say in competitive situations that backfire. For the Asana example: "Don't lead with price — Asana is seen as worth the premium by buyers who've already chosen it."

---

## Limitations

- Competitor research is from Tavily web results — public reviews and published pricing. Internal competitive intelligence from your own win/loss data should be added via `win_loss_context`.
- Battle cards go stale as competitors ship features. Regenerate quarterly or when significant competitor updates are announced.

---

## License

MIT.
