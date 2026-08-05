# database-migration-planner

A database migration that seems simple in a design doc ("rename this column") can lock a production table for minutes and take down the application if run naively. This plans migrations using expand-contract patterns to avoid downtime, sequences each phase to be independently safe and reversible, and includes specific verification and rollback steps for every stage — not just "run this SQL."

---

## What it does

Takes migration name, type, change description, current/desired schema, database engine, estimated row count, downtime tolerance, and dependent services. Claude produces:

- **Migration strategy** — expand_contract/blue_green/direct/batched_backfill with rationale
- **Risk level** — low/medium/high/critical with rationale
- **Phases** — each with: description, SQL or specific steps, reversibility flag, rollback steps for that specific phase, verification method, whether it's safe to pause after this phase, and estimated time
- **Deployment coordination** — application code changes that must deploy in sync with specific migration phases
- **Locking concerns** — specific to the database engine and table size
- **Backfill strategy** — how to batch data backfills to avoid locking or performance issues
- **Monitoring during migration** — specific metrics to watch
- **Go/no-go checklist** — items to verify before starting
- **Emergency rollback plan** — what to do if something goes wrong mid-migration
- **Post-migration cleanup** — steps after the migration is confirmed successful (e.g., dropping old columns)

HTML report with phase cards showing SQL in monospace blocks, reversibility badges, and verify/rollback side-by-side.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/plan-database-migration \
  -H "Content-Type: application/json" \
  -d '{
    "migration_name": "Rename assignee_id to assigned_user_id on tasks table",
    "migration_type": "column_rename",
    "database_engine": "postgresql",
    "table_name": "tasks",
    "estimated_row_count": "12 million rows",
    "downtime_tolerance": "zero_downtime",
    "read_write_volume": "~800 writes/sec peak, ~4000 reads/sec peak",
    "traffic_pattern": "Peak traffic 9am-6pm ET weekdays, low traffic overnight and weekends",
    "current_schema": "tasks(id, title, assignee_id, project_id, status, created_at, updated_at)",
    "desired_schema": "tasks(id, title, assigned_user_id, project_id, status, created_at, updated_at)",
    "dependent_services": ["task-api", "notification-service", "analytics-pipeline", "search-indexer"],
    "reply_email": "sara@flowdesk.com"
  }'
```

**Required:** `migration_name`, `change_description`

---

## Migration strategies

- **expand_contract** — add new structure alongside old, migrate reads/writes gradually, remove old structure last. Standard for zero-downtime renames, type changes, table splits.
- **blue_green** — build the new structure fully in parallel, cut over atomically. Used for larger structural changes.
- **direct** — single-step change, used only when downtime is acceptable or the table is small/low-traffic.
- **batched_backfill** — for large data migrations, process in batches to avoid long-running locks.

---

## Reversibility per phase

Every phase is explicitly marked reversible or not, with specific rollback steps for that phase alone — not just a single rollback plan for the whole migration. This matters because in a multi-phase migration, you often can't simply "undo everything" if you're partway through; you need to know how to safely stop and reverse from wherever you are.

---

## Limitations

- Generated SQL is a starting point based on the schema information provided — always review against your actual database engine's syntax and your ORM's migration tooling before running.
- This is a planning tool, not an execution tool. It doesn't run migrations — pair it with your existing migration framework (Rails migrations, Django migrations, Flyway, etc).
- For genuinely critical migrations on large production tables, always test the plan against a staging environment with production-like data volume first.

---

## License

MIT.
