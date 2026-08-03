# api-integration-tester

Verifying an API integration works usually means manually running a handful of curl commands and eyeballing the responses. This actually executes a batch of test scenarios against a live endpoint, checks status codes and expected response fields, then uses Claude to analyze failure patterns and give specific debugging guidance — not just "test 3 failed" but why it likely failed and what to check.

---

## What it does

Takes an integration name, endpoint base URL, and up to 15 test scenarios (each with method, path, request body, expected status, and expected response fields). The workflow:

1. **Executes each test** as an actual HTTP request against the live endpoint
2. **Evaluates the result** — checks status code match and whether expected fields are present in the response (supports dot notation for nested fields like `data.user.id`)
3. **Aggregates all results** and passes them to Claude for analysis

Claude then produces:
- **Test summary** — overall integration health
- **Failure analysis** — per failed test: likely cause, specific debugging steps, severity
- **Patterns observed** — commonalities across multiple failures (e.g., "all auth-related tests are failing, suggesting a token issue")
- **Integration readiness** — ready/needs_fixes/not_ready
- **Recommended next steps**

HTML report with pass/fail table, failure analysis cards with debugging steps, and pattern flags.

---

## Stack

n8n (HTTP Request node for actual test execution), Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/test-api-integration \
  -H "Content-Type: application/json" \
  -d '{
    "integration_name": "Notion Sync API",
    "endpoint_url": "https://api.staging.flowdesk.com",
    "auth_type": "bearer",
    "reply_email": "tom@flowdesk.com",
    "test_scenarios": [
      {"id": "T1", "name": "Get task list", "method": "GET", "path": "/api/v2/tasks", "expected_status": 200, "expected_fields": ["data", "data.tasks", "meta.total"], "description": "Basic task list retrieval"},
      {"id": "T2", "name": "Create task", "method": "POST", "path": "/api/v2/tasks", "request_body": {"title": "Test task", "project_id": "proj_test123"}, "expected_status": 201, "expected_fields": ["data.id", "data.title", "data.created_at"], "description": "Task creation"},
      {"id": "T3", "name": "Get nonexistent task", "method": "GET", "path": "/api/v2/tasks/nonexistent_id", "expected_status": 404, "expected_fields": ["error.message"], "description": "404 handling"},
      {"id": "T4", "name": "Bulk assign", "method": "POST", "path": "/api/v2/tasks/bulk-assign", "request_body": {"task_ids": ["t1", "t2"], "assignee_id": "usr_123"}, "expected_status": 200, "expected_fields": ["data.updated_count"], "description": "New bulk assignment endpoint"}
    ]
  }'
```

**Required:** `integration_name`, `endpoint_url`, `test_scenarios`

---

## How field checking works

`expected_fields` supports dot notation for nested response fields. `"data.user.id"` checks that the response has a `data` object containing a `user` object containing an `id` field. This catches both missing top-level fields and structural changes deep in the response.

---

## Actual execution, not simulation

Unlike agents that generate content, this one makes real HTTP requests to the endpoint you specify. Use this against staging or test environments, not production, unless the test scenarios are read-only and safe to run repeatedly.

---

## Auth

Pass `auth_type` and configure the actual auth header/token in your n8n HTTP Request node credentials — this workflow's template includes a Content-Type header by default; add your Authorization header in the node configuration for authenticated endpoints.

---

## Limitations

- Tests run sequentially against a live endpoint — this validates behavior, not load or performance.
- Field presence checking confirms structure but doesn't deeply validate field values or types. For value-level assertions, extend the evaluation logic in the "Evaluate Result" node.
- Up to 15 test scenarios per call to keep execution time reasonable.

---

## License

MIT.
