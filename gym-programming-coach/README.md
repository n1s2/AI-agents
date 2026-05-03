# gym-programming-coach

I've paid for online coaching before and the part that actually added value wasn't the check-in calls — it was getting a program written for me each week that adapted to how the previous week went. When I missed two sessions because of travel, the next week wasn't just a copy of the previous one.

This does the adaptive programming part. Every Sunday morning it reads your training history from the last 4 weeks, looks at consistency, checks your recent top loads on key lifts, reads any notes you left during sessions, and generates a full week of training. Day by day, exercise by exercise, with sets, reps, load guidance, rest periods, and coaching cues.

There's also a companion webhook for logging workout sets as you do them. Quick POST from a phone shortcut, data goes into the sheet, next Sunday's program reflects it.

---

## What it does

**Weekly program (every Sunday 7am):**
- Loads athlete profile: goal, training age, days available, session length, equipment, injuries, current lift benchmarks
- Loads last 4 weeks of workout data: session frequency per week, top loads by exercise, notes from last week
- Sends everything to Claude which writes a full week of programming adapted to the data
- Builds a formatted HTML email: coach's note, day-by-day sessions with warm-up/exercises/cool-down, weekly targets, nutrition note, next week preview
- Sends to your inbox

**Workout logging (webhook, any time):**
- POST a set: exercise, sets, reps, weight, RPE, notes
- Saves to Google Sheets instantly
- Returns confirmation

---

## Stack

- **n8n** — scheduler + webhook + workflow
- **Google Sheets** — profile + workout history
- **Anthropic Claude** (claude-opus-4-5) — program writing
- **SMTP** — email delivery

---

## Setup

### 1. Create the Google Sheet

Two tabs:

**Tab: Profile** — two columns: `setting` and `value`

| setting | value |
|---|---|
| primary_goal | hypertrophy |
| training_age | 3 years |
| days_per_week | 4 |
| session_length_minutes | 60 |
| equipment | full gym |
| injuries_limitations | left shoulder impingement — avoid overhead pressing |
| preferred_style | upper/lower split |
| current_lifts | Squat: 120kg x5, Bench: 90kg x5, Deadlift: 150kg x3 |

**Tab: Workouts** — columns:
```
date | exercise | sets | reps | weight_kg | weight_lbs | rpe | session_type | notes | logged_at
```
Leave it empty.

### 2. Environment variables

```
GYM_SHEET_ID=your_google_sheet_id
FROM_EMAIL=coach@yourdomain.com
ATHLETE_EMAIL=you@email.com
```

### 3. Credentials

- **Google Sheets OAuth2**
- **Anthropic API** (LangChain node)
- **SMTP**

### 4. Import and activate

Import `workflow.json`, activate. Test the workout logger first (see below), then manually trigger the Sunday scheduler to see your first program.

---

## Logging a workout set

```bash
curl -X POST https://your-n8n.com/webhook/log-workout \
  -H "Content-Type: application/json" \
  -d '{
    "date": "2025-04-29",
    "exercise": "Back Squat",
    "sets": 4,
    "reps": 5,
    "weight_kg": 115,
    "rpe": 8,
    "session_type": "Lower Strength",
    "notes": "Depth felt good, knees tracking well, could have done another rep on last set"
  }'
```

**Required:** `exercise`, `sets`, `reps`

Log one entry per exercise per session, or one per set if you want that granularity. The program generator aggregates by exercise anyway so either works.

---

## What Claude uses to adapt the program

- **Session frequency**: if you only trained 2 days last week instead of 4, Claude redistributes volume or acknowledges the disruption
- **Top loads**: used as a baseline for load guidance this week — if you hit 120kg×5 on squat, Claude programs around that rather than guessing
- **Notes**: if you wrote "right knee felt sore after lunges" in your workout notes, Claude sees that and can adjust programming or add a note to the affected exercises
- **Week-by-week trend**: 3 consistent weeks followed by 1 patchy week reads differently than 4 patchy weeks in a row

The `training_age` and `current_lifts` fields in your profile are the most important ones to fill out accurately. Claude uses these to set appropriate intensity ranges.

---

## Equipment options

The `equipment` field is freeform. Useful values:
- `full gym` — barbell, rack, cables, machines, everything
- `barbell and rack only`
- `dumbbells up to 40kg, pull-up bar, resistance bands`
- `home gym: barbell, rack, adjustable dumbbells`
- `bodyweight only`

Claude adapts exercise selection to whatever you specify. If you don't have cables, it won't program cable rows.

---

## Goals

The `primary_goal` field guides the overall program design. Common values:
- `strength` — lower rep ranges, longer rest, emphasis on big compound lifts
- `hypertrophy` — moderate reps, higher volume, more isolation work
- `fat loss` — circuit-friendly, supersets, conditioning emphasis
- `general fitness` — balanced across strength, conditioning, mobility
- `powerlifting` — specificity to squat, bench, deadlift
- `athletic performance` — explosiveness, movement quality, conditioning

---

## The RPE field

RPE (Rate of Perceived Exertion) on a 1–10 scale is optional but useful. If you consistently log RPE and it's high (8.5–9.5 every session), Claude will see that trend in the notes and may suggest a lighter week or deload. If RPE is consistently low (6–7), it may increase load guidance.

---

## Injuries and limitations

Be specific in the `injuries_limitations` field. "Left shoulder impingement" is more useful than "shoulder problems." Claude will avoid exercises that stress that area and may suggest substitutions. It's not a doctor — it won't diagnose you — but it will respect stated limitations in the programming.

---

## Deload weeks

Claude decides autonomously whether to program a deload based on training history. If you've had 4 hard weeks in a row with high RPE notes and consistent volume, it'll flag `"deload": true` in the program and reduce intensity/volume that week. You can also add a note in your profile: `deload_this_week: yes` and Claude will respect it.

---

## iOS shortcut for fast logging

I use a Shortcuts automation that prompts for exercise, weight, sets, reps, and RPE then fires the POST. Total time to log a set: under 20 seconds while the rest timer is running. The shortcut template isn't included yet but it's just a URL scheme with some input fields — straightforward to build.

---

## Limitations

- One athlete per workflow. For multiple people, duplicate the workflow and point each to a separate Google Sheet.
- Claude doesn't see your actual workout video or form — coaching cues are based on common form issues for each exercise, not your specific mechanics.
- The program is a starting point. If something doesn't feel right on a given day (different energy levels, equipment unavailable), trust your body and adjust.
- Load guidance uses your logged history. If you haven't logged any data yet, the first program will be based on profile alone and will be more conservative — that's intentional.

---

## Ideas

- [ ] Progress tracking charts (volume over time, lift PRs) via a monthly report
- [ ] In-session app: serve today's session as a JSON endpoint for a simple mobile UI
- [ ] Auto-deload trigger based on cumulative fatigue metrics
- [ ] Export to PDF or Apple Notes

---

## License

MIT.
