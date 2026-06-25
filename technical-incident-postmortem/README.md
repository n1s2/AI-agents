# technical-incident-postmortem

Postmortems written in the hours after a stressful incident are hard to write well. Engineers are tired, they're writing for multiple audiences (technical team, leadership, affected customers), and they need to be objective about events they were personally involved in. Most end up too short, blame-y, or full of vague action items that never get done.

This generates a complete blameless postmortem in Google/SRE style from a structured incident report: timeline, root cause hypothesis, impact, and resolution steps. Claude writes the narrative sections, identifies systemic contributing factors, and generates specific actionable action items with priority and owner.

---

## What it does

Takes incident details, a timestamped timeline, root cause hypothesis, and resolution steps. Claude generates:

- **Executive summary** — 3–4 sentences for non-technical stakeholders
- **What happened** — 3–5 paragraph narrative explaining the sequence of events and technical context
- **Root cause analysis** — immediate cause, contributing systemic factors, why it wasn't caught earlier
- **Impact assessment** — duration, users affected, services degraded, data loss, financial impact
- **Detection and response** — how the incident was found and how the response unfolded
- **What went well / poorly** — blameless, systemic framing
- **Action items** — specific, prioritized (P1/P2/P3), typed (prevention/detection/response/process), with owner and due date
- **Lessons learned** — broader takeaways for the system and team

HTML output with a full incident timeline table, color-coded severity badge, and action item table with priority indicators. Emails to team and/or reply email.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP.

---

## Setup

**Env vars:** `FROM_EMAIL`

**Credentials:** Anthropic API (LangChain node), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-postmortem \
  -H "Content-Type: application/json" \
  -d '{
    "incident_id": "INC-2025-047",
    "incident_title": "Payment API timeout causing checkout failures",
    "severity": "sev1",
    "incident_date": "2025-05-14",
    "oncall_engineer": "Tom Walsh",
    "incident_commander": "Priya Sharma",
    "postmortem_author": "Tom Walsh",
    "services_affected": ["checkout-service", "payment-api", "order-management"],
    "detection_method": "PagerDuty alert — error rate on /checkout exceeded 5% threshold",
    "incident_description": "The payment API began returning HTTP 504 timeouts at 14:32 UTC, causing checkout requests to fail. Approximately 34% of checkout attempts failed during the 47-minute window. Root cause was a database connection pool exhaustion caused by a slow query introduced in the 14:15 deployment.",
    "impact_description": "34% checkout failure rate for 47 minutes. Estimated 1,200 orders lost. No data loss. Revenue impact estimated at $48,000.",
    "root_cause_hypothesis": "Deployment at 14:15 introduced a new product recommendation query that runs at checkout. The query lacks a proper index and performs a full table scan on the products table (12M rows). Under normal load this runs in ~800ms but under peak checkout load (14:30-15:00 UTC daily peak) it caused connection pool exhaustion within 17 minutes.",
    "timeline": [
      {"time": "14:15 UTC", "actor": "CI/CD", "event": "v2.4.1 deployed to production"},
      {"time": "14:32 UTC", "actor": "PagerDuty", "event": "Alert: checkout error rate > 5%"},
      {"time": "14:34 UTC", "actor": "Tom Walsh", "event": "Acknowledged alert, began investigation"},
      {"time": "14:41 UTC", "actor": "Tom Walsh", "event": "Identified payment API timeout errors in logs"},
      {"time": "14:49 UTC", "actor": "Tom Walsh", "event": "Escalated to Priya as incident commander"},
      {"time": "14:52 UTC", "actor": "Priya Sharma", "event": "Opened incident bridge, assembled response team"},
      {"time": "15:03 UTC", "actor": "Amara Nwosu", "event": "Identified slow query in DB slow query log"},
      {"time": "15:11 UTC", "actor": "Tom Walsh", "event": "Initiated rollback of v2.4.1"},
      {"time": "15:19 UTC", "actor": "CI/CD", "event": "Rollback to v2.4.0 complete"},
      {"time": "15:21 UTC", "actor": "Priya Sharma", "event": "Error rate returned to baseline, incident resolved"}
    ],
    "resolution_steps": [
      "Identified slow query in database slow query log at 15:03",
      "Rolled back v2.4.1 to v2.4.0 at 15:11",
      "Verified error rate recovery at 15:21",
      "Added missing index as hotfix deployed at 16:45"
    ],
    "team_email": "engineering@company.com",
    "reply_email": "priya@company.com"
  }'
```

**Required:** `incident_title`, `incident_description`, `timeline` (array, min 2 events), `severity`

---

## Severity levels

`sev1` — critical (customer-facing outage, data loss risk)
`sev2` — major (significant degradation, major feature broken)
`sev3` — moderate (partial degradation, workaround available)
`sev4` — minor (cosmetic or low-impact issue)

Severity colors the badge and shapes the tone of the executive summary.

---

## Blameless framing

Claude is explicitly instructed not to blame individuals and not to accept "human error" as a root cause without asking why the system allowed the error. If someone "accidentally" deployed a bad query, the systemic question is: why wasn't this caught in code review, why didn't staging load tests surface it, why didn't the query plan check in CI catch it?

The contributing factors and what-went-poorly sections focus on process, tooling, and system design gaps — not who made a mistake.

---

## Action items

Each action item is:
- **Specific** — not "improve monitoring" but "add query execution time alert for queries > 500ms on the products table"
- **Typed** — prevention, detection, response, or process
- **Prioritized** — P1 (within 1 week), P2 (within 1 month), P3 (this quarter)
- **Owned** — assigned to a team or role
- **Dated** — suggested timeframe

Claude derives these from the root cause analysis — each contributing factor should map to at least one action item.

---

## Limitations

- The postmortem quality depends on the quality of input, especially the timeline and root cause hypothesis. A timeline with 2 vague events produces a thin narrative.
- Duration in `impact_assessment.duration_minutes` is inferred from the timeline if not explicitly stated — verify it against the actual incident record.
- The financial impact is a pass-through from `impact_description` — the agent doesn't calculate it independently.

---

## Ideas

- [ ] Incident tracking sheet: log all postmortems to a Sheet for trend analysis (recurring failure modes, MTTD/MTTR tracking)
- [ ] Jira integration: auto-create P1/P2 action items as tickets after postmortem generation
- [ ] PagerDuty trigger: connect PagerDuty webhook to auto-start a postmortem when a SEV1 incident is resolved
- [ ] Status page integration: pull the customer-facing incident summary from the postmortem executive summary automatically

---

## License

MIT.
