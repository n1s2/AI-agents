# api-documentation-generator

Writing API endpoint docs in Stripe/Twilio style — clear summary, parameter tables, realistic curl example, example request/response, error table — is repetitive once you've defined the endpoint shape. This generates the full doc page from a structured endpoint description.

---

## What it does

Takes endpoint path, method, description, auth type, path/query params, request/response schemas, and error codes. Claude writes a complete documentation page: title, summary, parameter tables, a realistic curl example, example success and error JSON responses consistent with any schema given, an error code table, implementation notes/gotchas, and a single markdown string ready to paste into a docs site. Builds an HTML preview (dark code blocks, method-colored badge) and emails it if requested.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-api-docs \
  -H "Content-Type: application/json" \
  -d '{
    "api_name": "Flowdesk",
    "base_url": "https://api.flowdesk.io/v1",
    "endpoint_path": "/projects/{project_id}/tasks",
    "http_method": "POST",
    "description": "Create a new task within a project",
    "auth_type": "Bearer token in Authorization header",
    "path_params": [{"name":"project_id","type":"string","required":true,"description":"UUID of the project"}],
    "request_body_schema": {"title":"string, required","assignee_id":"string, optional","due_date":"ISO 8601 date, optional"},
    "response_body_schema": {"id":"string","title":"string","status":"string","created_at":"ISO 8601"},
    "error_codes": [{"code":401,"meaning":"Unauthorized"},{"code":404,"meaning":"Project not found"}],
    "rate_limits": "100 requests/minute per API key",
    "reply_email": "devrel@flowdesk.io"
  }'
```

**Required:** `endpoint_path`, `http_method`, `description`

---

## No invented fields

Claude is instructed not to add fields beyond what's implied by the schemas you provide — the example payloads are generated to match your actual shape, not embellished with extra fields.

---

## License

MIT.
