# plant-care-reminder-agent

I have about 20 plants. I kill roughly 3 a year not from neglect exactly but from inconsistency — I water everything on the same day regardless of what each plant actually needs, forget which one got fertilized last, and when something starts looking wrong I either panic and overwater or ignore it hoping it'll fix itself.

This is two things in one workflow. The daily side fires every morning and checks which plants need water, fertilizer, or repotting based on their individual schedules logged in a Google Sheet. If anything is due, Claude writes a short warm care note with a specific tip for the most urgent plant, and it emails me. If nothing is due, it stays quiet.

The diagnosis side is a webhook. When something looks wrong — yellowing leaves, drooping, weird spots — I POST a description and Claude gives me a structured diagnosis: most likely cause with confidence level, exactly what to do today, what to monitor over the next week, what NOT to do (the common mistakes that make things worse), and recovery prognosis.

---

## What it does

**Daily care reminders (every day 8am):**
- Loads all plants from Google Sheets
- Calculates days since last watered, fertilized, repotted for each plant
- Compares against each plant's individual care schedule
- Identifies what needs doing today and what's coming up in 2-5 days
- If anything is due: Claude writes a short care note with a specific tip
- Sends a formatted email — only when there are actual tasks
- Silent days when nothing needs doing

**Problem diagnosis (webhook, on-demand):**
- POST: plant name, species, symptoms, environment context, last watered, pot type, light conditions
- Claude diagnoses: primary condition with confidence level, explanation tied to the specific symptoms, immediate action, 7-day plan, what not to do, secondary possibilities, recovery prognosis
- Returns full JSON diagnosis
- Optional: emails the diagnosis if `reply_email` is provided

---

## Stack

- **n8n** — scheduler + webhook + workflow
- **Google Sheets** — plant database with care schedules
- **Anthropic Claude** (claude-opus-4-5) — care notes + diagnosis
- **SMTP** — email delivery

---

## Setup

### 1. Create the Plants sheet

One tab: **Plants** — columns:

```
plant_name | species | location | pot_size | water_every_days | fertilize_every_days | repot_every_days | last_watered | last_fertilized | last_repotted | notes
```

Example rows:

| plant_name | species | location | pot_size | water_every_days | fertilize_every_days | repot_every_days | last_watered | last_fertilized | last_repotted | notes |
|---|---|---|---|---|---|---|---|---|---|---|
| Monstera | Monstera deliciosa | living room | 25cm | 7 | 21 | 365 | 2025-04-28 | 2025-04-01 | | indirect light |
| Snake Plant | Sansevieria | bedroom | 15cm | 14 | 30 | 730 | 2025-04-20 | 2025-03-15 | | almost no care needed |
| Fiddle Leaf | Ficus lyrata | corner | 20cm | 7 | 14 | 365 | 2025-04-27 | 2025-04-10 | | hates being moved |

### 2. Updating care dates

When you water a plant, update `last_watered` in the sheet to today's date. Same for fertilizing and repotting. This is the one bit of friction — you have to keep the sheet current.

I update it from a phone shortcut that opens the sheet directly to the plant's row. Some people use a small form. Whatever takes under 10 seconds.

### 3. Environment variables

```
PLANTS_SHEET_ID=your_google_sheet_id
FROM_EMAIL=plants@yourdomain.com
USER_EMAIL=you@email.com
```

### 4. Credentials

- **Google Sheets OAuth2**
- **Anthropic API** (LangChain node)
- **SMTP**

### 5. Import and activate

Import `workflow.json`, activate. The daily reminder fires at 8am. Test immediately by executing the daily scheduler manually — it won't send if nothing is due (which, on day one with all dates blank, everything will be due, so you'll get an email).

---

## Care schedule defaults

If a plant has no `water_every_days` set, it defaults to 7 days. Same defaults:
- `fertilize_every_days` → 30 days if blank
- `repot_every_days` → 365 days if blank

Set these per plant. Cacti might be `water_every_days: 21`. Ferns might be `3`. The system respects individual schedules, not a one-size-fits-all cadence.

---

## Using the diagnosis webhook

```bash
curl -X POST https://your-n8n.com/webhook/plant-problem \
  -H "Content-Type: application/json" \
  -d '{
    "plant_name": "Fiddle Leaf Fig",
    "species": "Ficus lyrata",
    "symptoms": "Lower leaves turning yellow then brown from the edges inward. Two leaves dropped this week. Top leaves look fine. Started about 10 days ago after I moved it to a new spot near the window.",
    "environment": "apartment, central heating, 20 degrees C",
    "last_watered": "5 days ago",
    "pot_type": "ceramic with drainage",
    "light_condition": "bright indirect",
    "reply_email": "me@email.com"
  }'
```

**Required:** `plant_name`, `symptoms`

The more context you give — environment, watering history, recent changes — the more accurate the diagnosis. "Leaves are yellow" is hard to diagnose. "Lower leaves yellowing from the edges, moved it last week, watered 3 days ago, ceramic pot, bright indirect light" gives Claude enough to be specific.

---

## What the diagnosis looks like

```json
{
  "plant_name": "Fiddle Leaf Fig",
  "primary_diagnosis": {
    "condition": "Transplant/relocation stress compounded by overwatering",
    "confidence": "high",
    "explanation": "The timing of symptoms matches the move exactly. Fiddle Leaf Figs are notoriously sensitive to positional changes — even a few feet can cause leaf drop. The edge-browning pattern before yellowing suggests the roots are stressed rather than the classic root rot pattern, which would show uniform yellowing. Watering 5 days after moving stressed roots is likely too soon.",
    "immediate_action": "Move it back as close to its original spot as possible. Do not water for another 5-7 days. Remove the two dropped leaves cleanly at the base.",
    "next_7_days": "Leave it completely alone except for observation. Expect 1-2 more leaves may drop as it adjusts. New growth at the top is a good sign."
  },
  "do_not_do": [
    "Do not mist the leaves — this increases disease risk without helping drought stress",
    "Do not fertilize while it's stressed — this burns already compromised roots"
  ],
  "recovery_prognosis": "good — Fiddle Leafs recover well from relocation stress if left alone and not overwatered during recovery"
}
```

---

## The daily email only sends when needed

If all your plants are on schedule, the workflow runs silently. No email. This is intentional — a daily email that's sometimes empty gets ignored fast. You only hear from it when there's something to do.

---

## Limitations

- Care schedules are based on fixed intervals, not environmental conditions. In summer your plants may need water twice as often as in winter. Adjust `water_every_days` seasonally or the reminders will drift out of sync.
- Updating the sheet after caring for a plant is manual. There's no automatic tracking — you tell it what you did.
- Diagnosis is based on text descriptions only. Visual inspection by an expert (or at minimum a photo) would always be more reliable. The diagnosis is good for the obvious stuff but not for subtle soil or root problems you can't see.

---

## Ideas

- [ ] Photo upload mode — pass image URL to Claude's vision for visual diagnosis
- [ ] Seasonal care adjustments — automatically lengthen watering intervals in winter
- [ ] Plant acquisition log — track when you bought each plant, where from, how much
- [ ] Care history page — searchable log of all watering and feeding events

---

## License

MIT.
