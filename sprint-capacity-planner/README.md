# sprint-capacity-planner

Sprint planning usually works backwards from commitment — teams agree to a sprint scope and then try to figure out if they have capacity for it. This works forward: given actual team availability (adjusted for PTO, meetings, and context-switching), backlog priorities, required skills, and blocked items, it recommends what should and shouldn't be in the sprint, assigns work based on skills, and surfaces risks before the sprint starts.

---

## What it does

Takes sprint name, duration, sprint goal, team members (with availability %, skills, PTO), backlog items (with story points, priority, required skills, blocked flag), and optional velocity history. The validator calculates raw effective capacity using a 70% efficiency factor (accounts for meetings, reviews, interruptions). Claude then:

- Selects recommended sprint items — each with skill-based assignment, rationale, and risk rating
- Defers low-priority or capacity-busting items with reasons
- Flags blocked items with unblocking recommendations
- Summarizes committed story points, estimated days, buffer, and utilization %
- Assesses sprint health: achievable/at-risk, confidence level, risk list, sprint goal coverage
- Shows per-member utilization bars with key items
- Writes facilitator notes for the sprint planning conversation

HTML report has item table, deferred list, blocked list, team utilization bars, and sprint risk flags.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/plan-sprint-capacity \
  -H "Content-Type: application/json" \
  -d '{
    "sprint_name": "Sprint 44",
    "sprint_duration_days": 14,
    "sprint_goal": "Ship Notion integration to production and resolve all P0 support tickets",
    "velocity_history": [42, 38, 45, 41, 36],
    "reply_email": "tom@flowdesk.com",
    "team_members": [
      {"name": "Tom Walsh", "role": "backend engineer", "availability_percent": 80, "planned_pto_days": 1, "skills": ["Node.js", "PostgreSQL", "API integrations", "Notion API"]},
      {"name": "Amara Nwosu", "role": "frontend engineer", "availability_percent": 100, "planned_pto_days": 0, "skills": ["React", "TypeScript", "UI components"]},
      {"name": "Sara Kim", "role": "backend engineer", "availability_percent": 60, "planned_pto_days": 0, "skills": ["Node.js", "PostgreSQL", "infrastructure", "monitoring"]},
      {"name": "Jeff Lin", "role": "fullstack engineer", "availability_percent": 100, "planned_pto_days": 2, "skills": ["React", "Node.js", "PostgreSQL"]}
    ],
    "backlog_items": [
      {"id": "FD-1201", "title": "Notion bidirectional sync — production release", "story_points": 8, "priority": "critical", "required_skills": ["Notion API", "Node.js"], "type": "feature"},
      {"id": "FD-1205", "title": "Notion OAuth reconnect flow edge cases", "story_points": 3, "priority": "high", "required_skills": ["Notion API", "React"], "type": "bugfix"},
      {"id": "FD-1189", "title": "Fix Safari iOS logout bug", "story_points": 2, "priority": "high", "required_skills": ["React"], "type": "bugfix"},
      {"id": "FD-1195", "title": "Bulk task assignment UI", "story_points": 5, "priority": "medium", "required_skills": ["React", "TypeScript"], "type": "feature"},
      {"id": "FD-1199", "title": "Email digest daily/weekly toggle", "story_points": 3, "priority": "medium", "required_skills": ["React", "Node.js"], "type": "feature"},
      {"id": "FD-1207", "title": "P0: enterprise SSO intermittent failure", "story_points": 5, "priority": "critical", "required_skills": ["Node.js", "infrastructure"], "type": "bugfix"},
      {"id": "FD-1210", "title": "Migrate analytics to new schema", "story_points": 8, "priority": "low", "required_skills": ["PostgreSQL"], "blocked": true, "type": "tech_debt"},
      {"id": "FD-1198", "title": "Notification preferences page", "story_points": 4, "priority": "low", "required_skills": ["React", "Node.js"], "type": "feature"}
    ]
  }'
```

**Required:** `sprint_name`, `sprint_duration_days`, `team_members`, `backlog_items`

---

## Capacity calculation

The validator calculates effective capacity as: `(sprint_days - pto_days) × (availability_percent / 100) × 0.7` per team member. The 0.7 factor accounts for meetings, code review, standups, and context-switching. This is conservative by design — sprint plans that assume 100% focus hours consistently fail.

---

## Velocity history

Pass your last 3–5 sprint velocities in `velocity_history` as an array of story point numbers. Claude uses this to sense-check whether the committed point total is realistic relative to historical performance.

---

## Blocked items

Items with `blocked: true` are separated from the main plan and given specific unblocking recommendations. They don't count toward committed capacity.

---

## Limitations

- Skill matching is best-effort based on the skills you provide for each team member. For specialized tasks with skill requirements not listed, Claude will note the gap.
- This is a planning guide, not a project management tool. It doesn't track progress during the sprint or update commitments automatically.

---

## License

MIT.
