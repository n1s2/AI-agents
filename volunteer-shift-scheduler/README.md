# volunteer-shift-scheduler

Coordinating volunteers manually is a spreadsheet nightmare. Someone emails their availability, another fills a Google Form, a third texts the coordinator directly. Then you sit down to build the schedule: cross-referencing availability, skill requirements, shift needs, and maximum hours per person — all by hand.

This handles the full cycle. Two webhooks: one for volunteers to submit their availability (with role, skills, preferred shifts, and max shifts per week), and one for the coordinator to trigger schedule generation. The matching algorithm assigns volunteers to shifts based on date availability, role, skill requirements, and workload limits. Claude summarizes what's staffed, what's not, and what to do about gaps. The coordinator gets an emailed schedule table.

Works for one-time events, recurring programs, and multi-day events.

---

## What it does

**Submit availability (POST `/volunteer-availability`):**
- Volunteer submits: name, email, available dates, role, skills, preferred shifts, max shifts per week
- Saves to Google Sheets Availability tab
- Sends confirmation email to volunteer
- Returns confirmation

**Generate schedule (POST `/generate-schedule`):**
- Coordinator submits: event name, list of shifts (each with date, time, role, volunteers needed, required skills)
- Loads all volunteer availability from Google Sheets
- Matching algorithm assigns volunteers to shifts based on:
  - Date availability match
  - Role match (or general volunteer for unspecified roles)
  - Required skills match
  - Max shifts per week not exceeded
- Claude writes a coordinator summary: staffing status, gaps, recommended actions
- Saves schedule to Google Sheets
- Emails coordinator the full schedule table
- Returns complete schedule JSON with all assignments

---

## Stack

- **n8n** — two webhooks
- **Google Sheets** — availability registry + schedule log
- **Anthropic Claude** (claude-sonnet-4-20250514) — schedule analysis
- **SMTP** — volunteer confirmation + coordinator schedule

---

## Setup

### 1. Create the Google Sheet

Three tabs:

**Tab: Availability** — columns:
```
volunteer_id | volunteer_name | volunteer_email | volunteer_phone | role | skills | available_dates | preferred_shifts | max_shifts_per_week | notes | submitted_at
```

**Tab: Schedule** — columns:
```
event_name | generated_at | total_shifts | fully_staffed | unfilled | schedule_json
```

**Tab: (optional) Volunteers** — a reference list if you want to pre-populate known volunteers

### 2. Environment variables

```
VOLUNTEER_SHEET_ID=your_google_sheet_id
FROM_EMAIL=volunteers@yourorg.org
```

### 3. Credentials

- **Google Sheets OAuth2**
- **Anthropic API** (LangChain node)
- **SMTP**

### 4. Import and activate

Import `workflow.json`, activate. Two webhook URLs will appear.

---

## Volunteer submitting availability

```bash
curl -X POST https://your-n8n.com/webhook/volunteer-availability \
  -H "Content-Type: application/json" \
  -d '{
    "volunteer_name": "Marcus Johnson",
    "volunteer_email": "marcus@email.com",
    "volunteer_phone": "555-0192",
    "role": "medical",
    "skills": ["first aid", "CPR", "triage"],
    "available_dates": ["2025-06-14", "2025-06-15", "2025-06-21"],
    "preferred_shifts": ["morning", "afternoon"],
    "max_shifts_per_week": 2,
    "notes": "Available from 7am on the 14th. Cannot do evenings."
  }'
```

**Required:** `volunteer_name`, `volunteer_email`, `available_dates`

---

## Generating a schedule

```bash
curl -X POST https://your-n8n.com/webhook/generate-schedule \
  -H "Content-Type: application/json" \
  -d '{
    "event_name": "Community Health Fair 2025",
    "event_description": "Annual free health screening event at Eastside Community Center",
    "notify_volunteers": true,
    "coordinator_email": "coordinator@healthfair.org",
    "reply_email": "coordinator@healthfair.org",
    "shifts": [
      {
        "shift_id": "shift-001",
        "date": "2025-06-14",
        "start_time": "08:00",
        "end_time": "12:00",
        "role": "medical",
        "volunteers_needed": 3,
        "required_skills": ["first aid"],
        "location": "Main Hall",
        "notes": "Must arrive 15 min early for briefing"
      },
      {
        "shift_id": "shift-002",
        "date": "2025-06-14",
        "start_time": "12:00",
        "end_time": "17:00",
        "role": "registration",
        "volunteers_needed": 4,
        "required_skills": [],
        "location": "Front entrance"
      },
      {
        "shift_id": "shift-003",
        "date": "2025-06-15",
        "start_time": "09:00",
        "end_time": "13:00",
        "role": "general",
        "volunteers_needed": 5,
        "required_skills": [],
        "location": "Various stations"
      }
    ]
  }'
```

**Required:** `event_name`, `shifts` (array with at least one shift)

---

## How matching works

For each shift the algorithm filters the volunteer pool to eligible candidates:

1. **Date match** — volunteer's `available_dates` must include the shift date (or contain it as a substring)
2. **Role match** — volunteer's role must match the shift role, OR either is `general`
3. **Skills match** — if the shift specifies `required_skills`, the volunteer must have all of them
4. **Workload limit** — volunteer's `assignedCount` must not exceed their `max_shifts_per_week`

Eligible volunteers are assigned in list order (first-come, first-served by submission date). The shift fills up to `volunteers_needed` and stops.

This is deterministic and transparent — no black-box optimization. For events where fairness of distribution matters more than availability ordering, add a shuffle before the slice or sort by `assignedCount` ascending to spread shifts evenly.

---

## Shift roles

Roles are free-text strings that must match between the volunteer's submitted role and the shift's `role` field. Convention:
- `general` — no specific role required, anyone can fill this
- `medical` — medically trained volunteers
- `registration` — front-of-house, check-in
- `logistics` — setup, teardown, equipment
- `translation` — language skills
- `driving` — transport volunteers

If a volunteer submits `role: "general"`, they're eligible for any shift regardless of the shift's role. If a shift has `role: "general"`, any volunteer is eligible regardless of their role.

---

## Required skills

`required_skills` on a shift must exactly match values in the volunteer's `skills` array (case-insensitive). Use consistent skill names across submissions and shift definitions.

Examples: `first aid`, `CPR`, `Spanish`, `forklift`, `food handling certificate`

---

## Unfilled shifts

If a shift can't be fully staffed from the available volunteer pool, it appears in the coordinator's summary with the current fill count. Claude's analysis highlights these gaps specifically and suggests next steps: reach out to specific volunteers not yet assigned, adjust shift requirements, or recruit additional volunteers.

---

## Notifying volunteers

`notify_volunteers: true` — this flag is captured and returned in the schedule JSON, but the current workflow doesn't automatically send individual shift assignments to volunteers. To add this:

After **Build Schedule Doc**, add a Code node that iterates through `assignments`, groups assignments by volunteer email, and sends each volunteer their assigned shifts. The data is all in the `assignments` array.

---

## Regenerating schedules

Submitting a new `/generate-schedule` request generates a fresh schedule from the current state of the Availability sheet. It doesn't modify or delete previous schedules in the Schedule tab — each generation is appended as a new row. To clear the availability pool before a new event, archive or delete rows from the Availability sheet.

---

## Limitations

- The matching algorithm is greedy and first-come-first-served within each eligibility filter. It doesn't optimize globally (e.g., minimizing total unmet demand) — it fills shifts sequentially in the order they appear in the `shifts` array.
- Date matching uses substring contains — "2025-06-14" in the available dates will match a shift with date "2025-06-14". Make sure dates are formatted consistently.
- Maximum shifts per week is enforced within a single schedule generation call. It doesn't check across multiple events or previous schedules.

---

## Ideas

- [ ] Volunteer notification: after generating, auto-send each assigned volunteer their specific shifts and shift details
- [ ] Waitlist mode: if a shift is overfilled by available volunteers, create a waitlist in case someone cancels
- [ ] Recurring availability: volunteers submit standing availability ("every Saturday morning") rather than specific dates
- [ ] Cancellation webhook: a third endpoint where a volunteer cancels, freeing their shifts for reassignment from the waitlist

---

## License

MIT.
