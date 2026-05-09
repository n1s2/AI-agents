# home-renovation-tracker

Renovations go wrong in two predictable ways: either things get forgotten and discovered late (that window was supposed to be moved), or there's no record of what happened when something goes wrong six months later (which contractor did the electrical?).

This is two tools in one workflow. The first is a project event logger — a webhook to POST milestones, payments, issues, delays, and decisions as they happen. Each event saves to Google Sheets with full context and returns a confirmation with the running spend total and active blocker count.

The second is a contractor brief generator. You describe the job, the trade, your house's quirks, your timeline, and your requirements. Claude writes a professional scope-of-work document structured specifically for that trade — what they need to bring, what's in and out of scope, how you'll know the job's done correctly, questions to answer in the quote, and anything they should know before showing up. Reduces the back-and-forth that wastes everyone's time.

---

## What it does

**Event logging (POST `/reno-project`):**
- Log: milestones, payments, issues, delays, decisions, inspections, quotes, completed work, material orders, notes
- Saves to Google Sheets with event type, date, amount, vendor, phase, blocker flag
- Loads project history and returns: total spent, active blockers, recent event feed
- Returns formatted HTML confirmation email (optional)

**Contractor briefing (POST `/reno-brief`):**
- Input: project scope, trade, house details, budget, timeline, specific requirements, materials preferences, access
- Claude writes a structured brief including: scope of work (must-have vs nice-to-have), out-of-scope list, site conditions, materials split (homeowner vs contractor), success criteria, questions to answer in quote, timeline expectations, warnings for the contractor
- Returns HTML brief + JSON
- Emails if `reply_email` provided

---

## Stack

- **n8n** — two webhooks + workflow
- **Google Sheets** — event log
- **Anthropic Claude** (claude-opus-4-5) — brief writing
- **SMTP** — optional email delivery

---

## Setup

### 1. Create the Google Sheet

One tab: **Events** — columns:
```
event_id | event_date | project_id | event_type | description | amount | vendor | phase | is_blocker | logged_at
```

### 2. Environment variables

```
RENO_SHEET_ID=your_google_sheet_id
RENO_CURRENCY=USD
FROM_EMAIL=reno@yourdomain.com
```

### 3. Credentials

- **Google Sheets OAuth2**
- **Anthropic API** (LangChain node)
- **SMTP** (optional)

### 4. Import and activate

Import `workflow.json`, activate. Two separate webhook URLs will appear — one for event logging, one for brief generation.

---

## Logging a project event

```bash
# Log a payment
curl -X POST https://your-n8n.com/webhook/reno-project \
  -H "Content-Type: application/json" \
  -d '{
    "project_id": "kitchen-2025",
    "event_type": "payment",
    "description": "First payment to Metro Tile — 50% deposit for backsplash installation",
    "amount": 1400,
    "vendor": "Metro Tile Co.",
    "phase": "tiling",
    "event_date": "2025-05-04"
  }'

# Log a blocker
curl -X POST https://your-n8n.com/webhook/reno-project \
  -H "Content-Type: application/json" \
  -d '{
    "project_id": "kitchen-2025",
    "event_type": "issue",
    "description": "Discovered old galvanized pipe behind the wall — plumber says it needs replacing before tile can go in. Adds 2-3 days.",
    "is_blocker": true,
    "event_date": "2025-05-04"
  }'
```

**Event types:** `milestone`, `issue`, `payment`, `decision`, `delay`, `inspection`, `quote_received`, `work_completed`, `material_ordered`, `note`

**Required:** `project_id`, `event_type`, `description`

---

## Generating a contractor brief

```bash
curl -X POST https://your-n8n.com/webhook/reno-brief \
  -H "Content-Type: application/json" \
  -d '{
    "project_id": "kitchen-2025",
    "project_name": "Kitchen Renovation 2025",
    "scope_description": "Remove existing subway tile backsplash and replace with zellige tile from floor to upper cabinets (approx 14 sq meters). Include new grout throughout. The existing tile is 2 layers deep in two sections — demo will require care around the window recess.",
    "contractor_trade": "tiler",
    "contractor_name": "Metro Tile Co.",
    "house_details": "1960s terrace house, plasterboard walls behind tile, existing waterproofing unknown — may need refreshing in the area behind the sink",
    "budget": "$2,800 supply and install",
    "timeline": "Need completed before May 20 — kitchen is in use until May 10",
    "specific_requirements": "Zellige tile supplied by homeowner (already purchased). Random staggered pattern, no tile cutters visible from front-facing view. Grout color: charcoal.",
    "materials_preference": "Mapei grout preferred. Waterproofing membrane if needed — homeowner to approve product before purchase.",
    "access_details": "Key under mat. Dog in backyard — keep gate closed. No access before 8am.",
    "reply_email": "homeowner@email.com"
  }'
```

**Required:** `project_id`, `project_name`, `scope_description`, `contractor_trade`

---

## What the contractor brief covers

Claude structures the brief specifically for the trade. A tiling brief looks different from a plumbing brief — it includes grout specifications, tile pattern preferences, and waterproofing concerns rather than pipe specs and pressure ratings. The more detail you put in, the more specific and useful the brief becomes.

The **"Please address in your quote"** section is particularly useful — Claude generates specific questions based on the scope that the homeowner genuinely needs answered before committing. Things like: "How will you handle the second tile layer behind the window recess?" or "Do you carry your own waterproofing membrane or should we source separately?"

The **"How we'll know it's done right"** section gives the contractor clear acceptance criteria — and gives the homeowner something concrete to assess the finished job against.

---

## Multiple projects

Each event has a `project_id` field. Use consistent IDs per project — `kitchen-2025`, `bathroom-reno`, `deck-build`. The event log tracks all projects in one sheet and the confirmation response filters to the relevant project's history and running spend.

---

## The blocker flag

Mark events as blockers when they stop work from proceeding — a missing inspection, a supplier delay, discovered conditions that require a design change. The confirmation response includes an `active_blockers` count so you always know the current project health at a glance.

Blockers don't auto-resolve. When a blocker is cleared, log a new event with `event_type: decision` or `milestone` and describe the resolution. Future versions will support a dedicated resolve endpoint.

---

## Limitations

- The brief generator doesn't know your specific house or the contractor's past work — it works with what you provide. The more detail in `house_details` and `specific_requirements`, the better the output.
- Event logging is manual. There's no integration with contractor invoicing platforms or calendar systems — those would be useful additions.
- The `is_blocker` field is set at logging time. There's no way to retroactively mark something as a blocker without editing the sheet directly.
- Photo URLs are stored in the sheet but not rendered anywhere in the current version. Add a Google Drive integration to auto-file photos if you want a visual record.

---

## Ideas

- [ ] Project dashboard: weekly summary email across all active projects — spend, blockers, recent events
- [ ] Quote comparison: submit multiple quotes for the same scope, Claude compares them
- [ ] Warranty tracker: log installation dates and warranty periods, alert before they expire
- [ ] Defect log: specific event type with photo evidence tracking for post-completion issues

---

## License

MIT.
