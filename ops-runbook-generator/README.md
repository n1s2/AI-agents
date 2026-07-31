# ops-runbook-generator

Runbooks written as documentation ("the process involves connecting to the database and checking the logs") aren't useful at 2am during an incident. A real runbook is instructions: numbered steps with specific commands, what to expect after each step, what to do if that expectation is wrong, a rollback procedure, and clear escalation criteria. This generates a complete operational runbook for any process type.

---

## What it does

Takes process name, type, description, trigger, prerequisites, tools required, common failure modes, and escalation path. Claude writes a complete runbook including:

- **Overview** — what this runbook covers, when to use it, expected outcome
- **Trigger and SLA** — what initiates this runbook and time constraint
- **Prerequisites** — each with how to verify it is met
- **Roles** — who does what
- **Steps** — each numbered step with: specific action (what to run/click/do), expected output (how you know it worked), failure handling (what to do if it doesn't), time estimate, and sub-steps
- **Verification** — checklist of checks to confirm success, pass criteria, sign-off requirement
- **Rollback procedure** — how to undo the process if something goes wrong
- **Common issues** — symptom, likely cause, resolution for known problems
- **Escalation** — specific conditions that require escalation, contacts with role and when to reach them
- **Post-completion actions** — notifications, tickets, documentation to update

HTML formatted as a clean runbook document with step cards (green success/red failure indicators), verification checklist, and escalation section.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-runbook \
  -H "Content-Type: application/json" \
  -d '{
    "process_name": "Emergency Database Failover — Primary to Read Replica",
    "runbook_type": "incident_response",
    "team_owner": "Platform Engineering",
    "primary_owner": "Sara Kim",
    "trigger": "Primary PostgreSQL instance is unresponsive for more than 5 minutes, confirmed by Datadog alert DB-PRIMARY-DOWN",
    "sla_or_time_limit": "Target: database connectivity restored within 15 minutes of alert",
    "escalation_path": "Sara Kim (primary) > Priya Sharma (VP Eng) > AWS Support",
    "review_cycle": "monthly",
    "tools_required": ["AWS RDS Console", "psql CLI", "Datadog", "PagerDuty", "Slack #incidents"],
    "prerequisites": [
      "AWS RDS console access with db-admin role",
      "psql installed and $DB_CONN_STRING env var set",
      "PagerDuty incident already opened"
    ],
    "common_failure_modes": "Read replica lag may be >5 minutes, causing data loss for recent writes. Multi-AZ failover may not trigger automatically if the health check is misconfigured. App connection pools may not reconnect automatically after failover.",
    "success_criteria": "All application health checks pass, database connection pool shows active connections to new primary, Datadog DB latency metrics return to normal range",
    "process_description": "When the primary RDS instance fails, promote the read replica to primary, update the connection string in AWS Secrets Manager, restart application servers to reconnect, and notify stakeholders. The read replica is in the same region. We use connection string rotation via Secrets Manager rather than direct RDS endpoint updates."
  }'
```

**Required:** `process_name`, `process_description`

---

## Runbook types

`incident_response`, `deployment`, `maintenance`, `onboarding`, `offboarding`, `escalation`, `data_backup`, `monitoring`, `release`, `routine_ops`

Type shapes what sections Claude emphasizes. Incident response runbooks lead with SLA and escalation. Deployment runbooks emphasize rollback. Maintenance runbooks emphasize scheduling and communication.

---

## Step failure handling

Every step includes a `failure_handling` field — what to do if the expected output doesn't appear or the step fails. This is what separates a runbook from a procedure document. An on-call engineer executing step 7 at 3am shouldn't have to guess what to do if step 7 produces an error.

---

## Rollback procedure

Claude always generates a rollback procedure — step-by-step instructions to undo the process. For deployments, this is a revert procedure. For database changes, this is a restore procedure. For incident response, this is a recovery validation checklist.

---

## Limitations

- Runbook specificity is proportional to process description detail. Vague descriptions produce structurally sound but generic runbooks. Detailed descriptions with specific commands, tools, and failure modes produce runbooks ready to execute.
- Commands, connection strings, and environment-specific details need to be added after generation. The runbook generates the structure and logic — specific values need to be filled in.

---

## License

MIT.
