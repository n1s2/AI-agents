# webhook-payload-validator

Webhook integrations break silently when the sending system changes its payload format — a field gets renamed, a type changes from string to number, a nested object flattens. By the time someone notices, days of data have been mangled. This runs real schema validation against sample payloads (checking field presence, type matching, and nested paths via dot notation), then uses Claude to diagnose failure patterns and distinguish schema drift from integration bugs from edge cases.

---

## What it does

Takes a schema name, expected schema definition, and up to 20 sample payloads (real webhook payloads you've received, with source and data). The workflow:

1. **Runs actual field validation** in a Code node — checks each expected field for presence, type match, and required status, supporting dot notation for nested fields
2. **Aggregates results** across all payloads
3. **Sends results to Claude** for pattern analysis

Claude then produces:
- **Validation summary** — overall payload health
- **Failure patterns** — grouped issues with affected payloads, likely cause (schema_drift/integration_bug/edge_case/sender_error), and recommendation
- **Schema recommendations** — how to make the schema more robust or accurate
- **Defensive handling suggestions** — code-level guidance for handling malformed payloads gracefully
- **Sender issues** — per source with specific issue and a suggested message to send that team
- **Overall readiness** — schema_ready/needs_adjustment/integration_broken

HTML report with validation table (valid/invalid per payload), failure pattern cards, and sender-specific issue cards with suggested outreach messages.

---

## Stack

n8n (Code node for actual validation logic), Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/validate-webhook-payload \
  -H "Content-Type: application/json" \
  -d '{
    "schema_name": "Stripe Payment Webhook",
    "reply_email": "platform@flowdesk.com",
    "expected_schema": {
      "fields": {
        "event_type": "string",
        "data.object.id": "string",
        "data.object.amount": "number",
        "data.object.currency": "string",
        "data.object.customer": "string",
        "created": "number"
      }
    },
    "sample_payloads": [
      {"source": "stripe_prod", "received_at": "2025-06-13T10:00:00Z", "data": {"event_type": "payment_intent.succeeded", "data": {"object": {"id": "pi_123", "amount": 1200, "currency": "usd", "customer": "cus_abc"}}, "created": 1718275200}},
      {"source": "stripe_prod", "received_at": "2025-06-13T10:05:00Z", "data": {"event_type": "payment_intent.succeeded", "data": {"object": {"id": "pi_124", "amount": "1200", "currency": "usd"}}, "created": 1718275500}},
      {"source": "stripe_test", "received_at": "2025-06-13T10:10:00Z", "data": {"event_type": "payment_intent.succeeded", "data": {"object": {"id": "pi_125", "amount": 500}}, "created": 1718275800}}
    ]
  }'
```

**Required:** `schema_name`, `expected_schema`, `sample_payloads`

---

## Schema format

```json
{
  "fields": {
    "field_name": "string",
    "nested.field.path": "number",
    "optional_field": {"type": "string", "required": false}
  }
}
```

Simple string values default to required. Use the object form to mark a field as optional.

---

## Dot notation for nested fields

`data.object.amount` checks that the payload has a `data` object containing an `object` containing an `amount` field. This catches issues at any depth in the payload structure, not just top-level fields.

---

## Distinguishing failure causes

Claude's analysis distinguishes:
- **schema_drift** — the sending system changed their format (needs coordination with them)
- **integration_bug** — your validation schema is wrong or outdated (needs a schema fix)
- **edge_case** — a legitimate variant your schema didn't account for (needs schema expansion)
- **sender_error** — the sending system has a bug producing malformed payloads (needs their fix)

Each gets a different recommended response.

---

## Limitations

- This validates structure (field presence and type), not business logic or value ranges. For deeper validation (e.g., "amount must be positive"), extend the validation logic in the "Run Schema Validation" node.
- Up to 20 sample payloads per call, sufficient for diagnosing patterns without needing your full payload history.

---

## License

MIT.
