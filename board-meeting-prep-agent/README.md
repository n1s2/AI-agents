# board-meeting-prep-agent

Walking into a board meeting without anticipating the hard questions means getting caught flat-footed defending a weak metric or fumbling a decision the board expected you to have already thought through. This helps founders prepare: anticipates likely hard questions with honest suggested answers, frames metrics talking points, identifies weak points to address proactively (before the board raises them), and preps decision recommendations with the strongest counter-argument the founder should expect.

---

## What it does

Takes company name, meeting date, key metrics, board members, agenda topics, decisions needed, known concerns, and previous action items. Claude produces:

- **Meeting narrative** — the story arc the founder wants the board to understand by the end
- **Suggested agenda order** — with rationale for sequencing
- **Metrics talking points** — each metric with framing (trend, comparison, driver) and an honest answer if the board pushes on it
- **Weak points to address proactively** — specific framing to raise issues before the board does, showing awareness and a plan
- **Anticipated hard questions** — likely questions with why the board will ask, an honest suggested answer, and what supporting data to have ready
- **Decision prep** — for each decision needed: recommendation, rationale, and the strongest counter-argument to expect
- **Previous action item status** — suggested updates, including honest framing for items not fully resolved
- **Pre-read summary** — 150-word summary for the board pre-read email
- **Closing ask** — what specifically to ask the board for

HTML document with color-coded sections: red for hard questions, amber for weak points, blue for decisions.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/prep-board-meeting \
  -H "Content-Type: application/json" \
  -d '{
    "company_name": "Flowdesk",
    "meeting_date": "2025-06-20",
    "quarter_or_period": "Q2 2025",
    "board_members": ["Maria Chen (Northbeam Ventures)", "David Park (Foundry Group)", "Jake Reyes (CEO)"],
    "company_context": "Series A company, 45 people, $1.2M ARR, 9 months post-raise",
    "reply_email": "jake@flowdesk.com",
    "key_metrics": {
      "MRR": "$102k (+4.2% MoM, down from 8% MoM last quarter)",
      "Net revenue retention": "108%",
      "Gross margin": "78%",
      "Burn rate": "$185k/month",
      "Runway": "14 months",
      "CAC payback": "11 months, up from 8 months"
    },
    "agenda_topics": [
      "Q2 metrics review",
      "Hiring plan for H2",
      "Series B timing discussion",
      "Competitive positioning update"
    ],
    "big_decisions_needed": [
      "Approve H2 hiring plan (6 new hires, $840k additional annual burn)",
      "Timing for Series B conversations - now vs Q4"
    ],
    "known_concerns": "Growth rate has decelerated from 8% to 4.2% MoM over the last quarter. CAC payback has also increased. We believe this is due to a pricing change we made in April that we are now reversing, but the board has not seen this data yet.",
    "previous_action_items": [
      "Hire VP of Sales - not yet completed, in final round with 2 candidates",
      "Improve gross margin to 75%+ - completed, now at 78%",
      "Reduce customer concentration risk - top customer now 8% of revenue, down from 15%"
    ]
  }'
```

**Required:** `company_name`, `meeting_date`, `key_metrics`

---

## Honest, not defensive

Claude is instructed to be direct about weak points, not just cheerleading. For the growth deceleration in the example, the prep doesn't suggest spinning the number — it suggests proactively raising it with the pricing change context before the board asks, which is a stronger position than waiting to be asked and appearing defensive.

---

## Decision prep includes the counter-argument

For each decision needed, Claude prepares not just the recommendation but the strongest counter-argument a board member might raise. This means the founder walks in having already thought through the pushback, rather than being surprised by it in the room.

---

## Metrics framing vs metrics spin

The `metrics_talking_points` section frames numbers honestly — trend context, what drove the change — while the `if_asked_why` field gives the honest answer if pushed further. This is framing, not spin: the goal is a founder who can discuss the number confidently because they've already thought it through, not one who's hiding something.

---

## Limitations

- This is prep material, not a replacement for genuinely understanding your own numbers. The founder should still deeply know their metrics — this helps structure the narrative and anticipate questions.
- Board dynamics vary — adjust tone and directness based on your actual board relationship.

---

## License

MIT.
