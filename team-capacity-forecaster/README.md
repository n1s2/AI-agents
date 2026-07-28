# team-capacity-forecaster

Committing to a project roadmap without understanding actual team capacity leads to missed deadlines, burnout, and credibility damage with stakeholders. This takes your team's availability (including leave, part-time, and ramp-up time), your project backlog, and existing commitments — then forecasts capacity over a 2–26 week horizon with honest assessments of what fits, what's at risk, and where skills gaps will create problems.

---

## What it does

Takes team members (with utilization %, planned leave, skills, current load), projects (with estimates, priorities, required skills, start week), and existing commitments. The validator auto-calculates total effective capacity using a 75% efficiency factor. Claude produces:

- **Capacity summary** — total available/committed/project weeks, buffer, utilization %, overall status (comfortable/tight/overcommitted)
- **Project fit analysis** — each project with feasibility (fits/tight/at_risk/wont_fit), recommended start week, estimated completion week, specific resource conflicts, notes
- **Member utilization** — weeks available/committed and utilization % per person, overloaded weeks flagged
- **Skills gap analysis** — each gap with severity (critical/moderate/minor), which projects need it, who has it, and mitigation
- **Risk flags** — specific risks with impact level and mitigation
- **Recommendations** — prioritized as immediate/this_quarter/nice_to_have

HTML report with status badge, project feasibility table, member utilization bars with overload flags, gap cards, and recommendation list.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/forecast-team-capacity \
  -H "Content-Type: application/json" \
  -d '{
    "team_name": "Platform Engineering",
    "forecast_period_weeks": 12,
    "forecast_start_date": "2025-07-07",
    "reply_email": "priya@flowdesk.com",
    "team_velocity_notes": "Team typically delivers 85% of committed work per sprint. We run 2-week sprints.",
    "existing_commitments": "On-call rotation (1 engineer at 20% at all times). Weekly 3-hour all-team meeting. Tom is mentoring a junior engineer at 5% time.",
    "hiring_plan": "One additional backend engineer joining in week 6",
    "team_members": [
      {"name": "Tom Walsh", "role": "Senior Backend", "utilization_percent": 80, "planned_leave": ["week 4", "week 5"], "skills": ["Node.js", "PostgreSQL", "integrations"], "current_load": "heavy"},
      {"name": "Amara Nwosu", "role": "Frontend", "utilization_percent": 100, "skills": ["React", "TypeScript", "design systems"], "current_load": "normal"},
      {"name": "Sara Kim", "role": "Backend", "utilization_percent": 100, "skills": ["Node.js", "PostgreSQL", "infrastructure"], "current_load": "normal"},
      {"name": "Jeff Lin", "role": "Fullstack", "utilization_percent": 100, "planned_leave": ["week 8", "week 9"], "skills": ["React", "Node.js", "PostgreSQL"], "current_load": "normal"},
      {"name": "New Hire", "role": "Backend", "utilization_percent": 50, "start_date": "week 6", "skills": ["Node.js"], "current_load": "ramping"}
    ],
    "projects": [
      {"name": "Email digest feature", "priority": "high", "estimated_weeks": 3, "required_skills": ["React", "Node.js"], "start_week": 1, "status": "planned"},
      {"name": "Onboarding flow redesign", "priority": "high", "estimated_weeks": 4, "required_skills": ["React", "TypeScript"], "start_week": 1, "status": "planned"},
      {"name": "Analytics v2 migration", "priority": "medium", "estimated_weeks": 6, "required_skills": ["PostgreSQL", "Node.js"], "start_week": 4, "status": "planned"},
      {"name": "Slack integration", "priority": "medium", "estimated_weeks": 3, "required_skills": ["Node.js", "integrations"], "start_week": 6, "status": "planned"},
      {"name": "Mobile app v1", "priority": "low", "estimated_weeks": 8, "required_skills": ["React", "Node.js"], "start_week": 8, "status": "planned"}
    ]
  }'
```

**Required:** `team_name`, `forecast_period_weeks`, `team_members`

---

## 75% efficiency factor

The validator applies a 75% efficiency factor to all available time. A team member available 100% for 12 weeks doesn't produce 12 person-weeks of output — meetings, code review, interruptions, and context-switching consume the rest. This factor is applied before Claude analyzes, so the feasibility assessments reflect realistic capacity, not optimistic capacity.

---

## Skills gap analysis

Claude maps each project's required skills to team members who have them. If a project needs `integrations` and only one person has that skill, and that person is also on leave during the planned window, that surfaces as a critical gap with a specific mitigation suggestion (e.g., "cross-train Sara on integrations in week 2").

---

## New hire ramp-up

Pass new team members with `start_date` (week number) and a reduced `utilization_percent` (50% is typical for someone ramping). Claude accounts for their delayed availability and reduced output in the forecast.

---

## Limitations

- The forecast is only as accurate as the estimates provided. For teams without historical velocity data, estimates will be imprecise — treat the feasibility assessments as directional.
- This forecasts capacity, not project outcomes. External dependencies, scope changes, and unexpected technical complexity aren't modeled.

---

## License

MIT.
