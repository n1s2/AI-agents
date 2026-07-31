# sales-objection-trainer

Generic objection handling advice ("acknowledge, respond, close") doesn't help a rep who just heard "we already have a solution for this" from a VP mid-demo. This generates a specific playbook for a specific objection at a specific deal stage: diagnosing what the prospect probably really means, an acknowledge-isolate-respond-advance framework with exact language, what not to say (with alternatives), scenario drills, and a coaching note on the underlying skill gap.

---

## What it does

Takes a specific objection verbatim, product name, objection type, deal stage, prospect title, and company/competitive context. Claude produces:

- **Objection diagnosis** — surface objection vs likely real concern, how to tell if it's the real blocker vs a brush-off, deal risk (high/medium/low), urgency
- **Response playbook** — four-step framework with exact language:
  1. **Acknowledge** — specific script that makes the prospect feel heard without agreeing
  2. **Isolate** — question to confirm this is the real objection, what to listen for
  3. **Respond** — multiple approaches (reframe, evidence, compromise, explore) each with specific script and when to use it
  4. **Advance** — how to move forward, what the next step should be
- **What not to say** — specific phrases to avoid, why they backfire, what to say instead
- **Scenario drills** — 3–5 prospect says / rep says pairs with why it works
- **When to walk away** — specific signals this deal is not going to close
- **Coaching note** — the underlying skill this rep should practice for this objection class

HTML training card with color-coded playbook sections (blue/purple/green/amber), drill cards with prospect/rep dialog, and avoid/instead pairs.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/train-objection-response \
  -H "Content-Type: application/json" \
  -d '{
    "objection": "We already use Asana and our dev team loves it. I would have to convince 40 people to switch and I just do not have the bandwidth for that right now.",
    "product_name": "Flowdesk",
    "objection_type": "competitor",
    "deal_stage": "demo",
    "prospect_title": "VP Operations",
    "rep_name": "Jake Reyes",
    "company_context": "80-person logistics company, Asana in use by dev team (12 people), ops team (22 people) still using spreadsheets. We talked to ops not IT.",
    "product_value_prop": "Flowdesk is built for ops teams, not developers. No IT involvement, 1-hour setup, replaces spreadsheets without requiring a company-wide migration.",
    "win_loss_context": "We have won deals where Asana is present by positioning as the ops team tool alongside Asana for dev. We have lost when the decision maker conflates Asana adoption with a company-wide mandate.",
    "competitor_context": "Asana is widely adopted in dev teams. Ops teams rarely love it — they find it complex. The migration fear is usually the real blocker, not satisfaction with Asana.",
    "reply_email": "jake@flowdesk.com"
  }'
```

**Required:** `objection`, `product_name`

---

## Objection types

`price`, `competitor`, `timing`, `authority`, `need`, `trust`, `technical`, `contract`, `internal_process`, `feature_gap`

Type calibrates the diagnosis and response approach. Price objections need ROI reframing. Competitor objections need displacement or coexistence plays. Timing objections need urgency creation without pressure. Authority objections need a mapping conversation to find the real decision maker.

---

## Deal stage calibration

The same objection means different things at different stages. "We need to think about it" at discovery is different from "we need to think about it" at proposal. Claude calibrates the response to the stage — earlier stages get exploratory responses, later stages get more direct closes.

---

## Scenario drills

The `scenario_drills` section gives the rep 3–5 specific exchanges to practice: what the prospect might say (variations on the objection) and exactly what the rep should say, with the reasoning explained. These are designed for role-play practice, not just reading.

---

## The isolate step

The most skipped step in objection handling. Claude always generates an isolation question — a way to confirm this is the real objection before spending time responding to something that isn't actually the blocker. "Is that the main concern, or is there something else holding you back?" sounds simple but changes the conversation.

---

## Limitations

- Training is for one specific objection per call. For a full objection library, call the agent once per objection and collect the training cards. For high-volume teams, build a workflow that generates a card for each objection in your CRM's loss reason data.
- Scripts are starting points — reps should adapt them to their own voice rather than reading them verbatim.

---

## License

MIT.
