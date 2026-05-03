# tenant-maintenance-request-router

Managing maintenance requests across multiple units is a logistics problem that scales badly. A tenant sends a WhatsApp message at 11pm about a leak. Is it a drip or is it flooding? Which trade does it need? Is it an emergency or can it wait until Monday? The property manager has to figure all of this out, often from a vague three-word description.

This takes a tenant's maintenance request, runs it through Claude which categorizes it, assesses urgency, identifies the right trade, flags safety risks, writes notes for whoever's showing up, and drafts a tenant acknowledgement — then routes it correctly. Emergencies get a red-header alert to the manager. Standard requests get a structured email. The tenant gets an acknowledgement if they provided an email. Everything logs to Google Sheets.

The triage is the useful part. "Water coming through ceiling" correctly flags as plumbing emergency with a safety risk note. "Lightbulb in bathroom needs replacing" comes in as low-urgency handyman. The manager spends time on decisions, not categorization.

---

## What it does

1. Accepts a POST: tenant name, unit, description, optional category hint, photo URLs, entry permission, preferred time slot
2. Validates and generates a ticket ID
3. Sends to Claude which triages: category, urgency (emergency/high/medium/low), safety risk, assigned trade, scope estimate, tenant acknowledgement text, internal notes for the trade, clarifying questions if the description is ambiguous
4. Saves everything to Google Sheets
5. Routes based on urgency:
   - **Emergency**: red-header alert email to manager, dispatched immediately
   - **High/Medium/Low**: structured standard alert to manager
6. If tenant provided email: sends them the Claude-written acknowledgement with ticket ID
7. Returns JSON confirmation to the webhook caller

---

## Stack

- **n8n** — webhook + workflow
- **Anthropic Claude** (claude-opus-4-5) — triage and communication drafting
- **Google Sheets** — request log
- **SMTP** — email routing

---

## Setup

### 1. Create the Requests sheet

One tab: **Requests** — columns:

```
ticket_id | submitted_at | tenant_name | tenant_email | tenant_phone | unit | building | description | category | urgency | safety_risk | assigned_trade | estimated_scope | entry_permission | preferred_time | status | internal_notes
```

### 2. Environment variables

```
MAINTENANCE_SHEET_ID=your_google_sheet_id
DEFAULT_BUILDING=Riverside Apartments
FROM_EMAIL=maintenance@yourproperty.com
MANAGER_EMAIL=manager@yourproperty.com
```

### 3. Credentials

- **Google Sheets OAuth2**
- **Anthropic API** (LangChain node)
- **SMTP**

### 4. Import and activate

Import `workflow.json`, activate, copy the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/maintenance-request \
  -H "Content-Type: application/json" \
  -d '{
    "tenant_name": "Marcus Webb",
    "tenant_email": "marcus@email.com",
    "tenant_phone": "+1 555 0192",
    "unit": "3C",
    "building": "Riverside Apartments",
    "description": "Hot water stopped working completely this morning. Cold water is fine from all taps. Boiler is making a clicking noise when it tries to start. No hot water for shower or kitchen. Has been 6 hours.",
    "preferred_time_slot": "morning",
    "entry_permission": true,
    "photo_urls": []
  }'
```

**Required:** `tenant_name`, `unit`, `description`

**Optional:** `tenant_email`, `tenant_phone`, `building`, `category` (hint for Claude), `photo_urls` (array of strings), `preferred_time_slot`, `entry_permission` (boolean, defaults to true)

---

## Example triage output

For the request above, Claude would produce something like:

```json
{
  "category": "hvac",
  "urgency": "high",
  "urgency_reason": "Complete loss of hot water for 6+ hours affects habitability.",
  "safety_risk": false,
  "assigned_trade": "hvac_tech",
  "estimated_scope": "investigation_needed",
  "tenant_acknowledgement": "Hi Marcus, we've received your maintenance request (MR-1745923847) about the hot water outage in unit 3C. This has been flagged as high priority and we'll have an HVAC technician contact you within 24 hours to arrange access. Thank you for reporting this promptly.",
  "internal_notes": "Boiler clicking on ignition attempt, no hot water from any tap for 6h. Cold supply intact. Likely ignition failure or gas valve issue — bring igniter components. Tenant available mornings.",
  "suggested_questions": []
}
```

---

## Building a tenant-facing form

The webhook is just a POST endpoint. Pair it with:

- **Tally.so**: free, no-code form builder with native webhook support. Looks professional, works on mobile. Takes 10 minutes to set up.
- **Typeform**: more polished, also has webhook integration
- A simple HTML form hosted anywhere

Map form fields to the webhook body. The `description` field should be a long-text textarea — the richer the description, the better the triage.

---

## Multi-building routing

If you manage multiple buildings with different managers, add a `building_manager_email` lookup. In the **Validate & Enrich** node, map building name → manager email from a config object:

```js
const managerMap = {
  'Riverside Apartments': 'riverside-manager@company.com',
  'Oak Street Complex': 'oak-manager@company.com'
};
const managerEmail = managerMap[body.building] || $env.MANAGER_EMAIL;
```

Then pass `managerEmail` through to the alert nodes.

---

## Trades and who they map to

Claude assigns one of: `plumber`, `electrician`, `hvac_tech`, `handyman`, `pest_control`, `locksmith`, `building_manager`, `cleaning_crew`, `contractor`.

You can extend the routing by adding a lookup node after the triage that maps trade → contractor contact info (phone, email) and includes that in the manager alert. This is left intentionally simple since trade contacts vary by property.

---

## Emergency threshold

Claude classifies as emergency when the description implies: active flooding, gas smell, total power loss, fire risk, security breach, sewage backup, or complete loss of essential services (heat in winter, etc.). The system prompt uses `temperature: 0.2` intentionally — low creativity, high consistency.

If you want to override or adjust what counts as emergency, edit the system prompt in the **Claude Triage Engine** node. The urgency is a judgment call; Claude errs toward caution, which is the right default.

---

## Photo URLs

The `photo_urls` field accepts up to 4 URLs. These are saved to the sheet but not currently passed to Claude (the free Anthropic API tier doesn't support vision in all configurations). If you're on a tier that supports image input, extend the **Claude Triage Engine** node to include the images in the message content — Claude's triage improves significantly when it can see the actual problem.

---

## Updating ticket status

When a job is completed, update the `status` column in the sheet to `resolved`. Build a second simple webhook (`/resolve-ticket`) that accepts a `ticket_id` and updates that row if you want to close tickets programmatically.

---

## Limitations

- Triage accuracy depends on description quality. "It's broken" produces a weak triage. The form should prompt tenants to describe what's happening, when it started, and whether it's getting worse.
- Claude doesn't know your specific contractors or their availability. The triage assigns a trade type, not a specific person.
- No SLA tracking built in yet. The sheet has the timestamps — a separate daily report could flag tickets that have been open too long.

---

## Ideas

- [ ] Resolve ticket endpoint + SLA breach alerting
- [ ] WhatsApp intake (Twilio webhook → this workflow)
- [ ] Per-trade email routing (send the internal notes directly to the assigned contractor)
- [ ] Tenant portal to check ticket status

---

## License

MIT.
