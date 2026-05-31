# api-health-monitor

Built this because I kept finding out services were down from users, not from monitoring. A five-minute check interval means the worst case is five minutes of downtime before the team knows. That's not great, but it's a lot better than "we found out when someone tweeted."

This pings every endpoint in a Google Sheet every 5 minutes, tracks response status and time, counts consecutive failures, and alerts via email and Slack when something goes down — and again when it recovers. Each endpoint has its own failure threshold (default 2 consecutive failures before alerting, to avoid noise from transient errors).

A companion webhook lets you add new endpoints without touching the sheet.

---

## What it does

**Monitor loop (every 5 minutes):**
- Loads all enabled endpoints from Google Sheets
- For each endpoint: makes the HTTP request with the configured method, auth header, and timeout
- Evaluates: is the response status what we expected? How fast was it?
- Updates the endpoint's current status and fail count in the sheet
- Logs every check to an Incidents tab
- If `consecutiveFailsBeforeAlert` threshold is crossed: sends email + Slack alert
- If a previously down endpoint comes back up: sends recovery alert

**Add endpoint (webhook `/add-endpoint`):**
- POST URL, name, method, expected status, timeout, auth header, alert destinations
- Adds directly to the Endpoints sheet
- Returns confirmation

---

## Stack

- **n8n** — 5-minute scheduler + webhook
- **Google Sheets** — endpoint registry + incident log
- **SMTP** — alert emails
- **Slack** — alert messages

---

## Setup

### 1. Create the Google Sheet

Two tabs:

**Tab: Endpoints** — columns:
```
name | url | method | expected_status | timeout_ms | auth_header | alert_email | alert_slack | fails_before_alert | enabled | current_fail_count | last_status | last_checked | response_time_ms | added_at
```

**Tab: Incidents** — columns:
```
timestamp | endpoint_name | url | status | response_time_ms | fail_count | event
```

Fill in your endpoints. Minimum required: `name`, `url`.

### 2. Environment variables

```
MONITOR_SHEET_ID=your_google_sheet_id
FROM_EMAIL=alerts@yourcompany.com
DEFAULT_ALERT_EMAIL=ops@yourcompany.com
DEFAULT_SLACK_CHANNEL=#alerts
```

### 3. Credentials

- **Google Sheets OAuth2**
- **SMTP**
- **Slack API**

### 4. Import and activate

Import `workflow.json`, activate. The monitor starts on the next 5-minute interval. Test by adding an endpoint and watching the Incidents tab fill in.

---

## Adding an endpoint via webhook

```bash
curl -X POST https://your-n8n.com/webhook/add-endpoint \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Payments API",
    "url": "https://api.yourservice.com/health",
    "method": "GET",
    "expected_status": 200,
    "timeout_ms": 8000,
    "auth_header": "Bearer your-token-here",
    "alert_email": "payments-team@company.com",
    "alert_slack": "#payments-alerts",
    "fails_before_alert": 2
  }'
```

**Required:** `url`

---

## Endpoint sheet columns

| Column | Default | Notes |
|---|---|---|
| `name` | url | Display name for alerts |
| `url` | required | Full URL including protocol |
| `method` | GET | GET, POST, HEAD |
| `expected_status` | 200 | HTTP status code expected |
| `timeout_ms` | 10000 | Milliseconds before timeout |
| `auth_header` | empty | Authorization header value |
| `alert_email` | default | Override alert destination per endpoint |
| `alert_slack` | default | Override Slack channel per endpoint |
| `fails_before_alert` | 2 | Consecutive failures before alerting |
| `enabled` | TRUE | Set to FALSE to pause monitoring |

---

## Alert thresholds

The `fails_before_alert` column controls how many consecutive failures trigger an alert. Default is 2 — this absorbs transient errors (a single slow request, a brief network blip) without waking anyone up.

For critical payment APIs or auth services, set to 1 — any failure is worth knowing about immediately.

For lower-priority endpoints, set to 3 or 4.

---

## Recovery alerts

When an endpoint that was down comes back up, a recovery alert fires automatically — both email and Slack. The recovery message says how many failed checks there were before recovery. This closes the loop: the team knows the incident is resolved without having to check manually.

---

## Per-endpoint alert routing

Each endpoint can have its own `alert_email` and `alert_slack` — useful when different endpoints are owned by different teams. The payments team gets payments API alerts, the auth team gets auth service alerts. If left blank, alerts go to `DEFAULT_ALERT_EMAIL` and `DEFAULT_SLACK_CHANNEL`.

---

## Pausing monitoring

Set `enabled` to `FALSE` in the sheet to pause monitoring for a specific endpoint. Useful during planned maintenance — prevents alert noise when you're intentionally taking something down.

---

## Check interval

Default is 5 minutes. Change `minutesInterval` in the **Every 5 Minutes** scheduler node. For critical services where 5 minutes is too slow, change to 1 or 2. For low-priority endpoints, push to 15 or 30.

Note: more frequent checks = more Google Sheets API calls. For large endpoint lists at 1-minute intervals, consider switching the storage to a lightweight database.

---

## The Incidents tab

Every check is logged — successful or not. This gives you a history of uptime, response times, and incident timelines. Useful for postmortems ("the API was first slow at 14:23, first returned a 500 at 14:28, full outage by 14:31").

Filter the Incidents tab by `event = ALERT` to see only incidents, or by `event = RECOVERED` to see resolution times.

---

## Authenticated endpoints

The `auth_header` column stores the full Authorization header value — e.g. `Bearer your-api-key` or `Basic base64encodedcredentials`. This is sent as the `Authorization` header on every check.

For OAuth tokens that expire, you'd need to add a token refresh step before the check loop. The current version assumes static credentials.

---

## Limitations

- Checks are sequential per endpoint, not parallel. For large endpoint lists (50+), the 5-minute cycle might be tight. Add a Batch/Parallel node to check multiple endpoints simultaneously.
- The `auth_header` value is stored in plaintext in Google Sheets. For production use, store sensitive tokens in n8n credentials or environment variables and reference them by name rather than storing the values in the sheet.
- Status codes are compared exactly. A 201 when you expect 200 will trigger a failure. Make sure `expected_status` matches what your endpoint actually returns under normal operation.
- Timeouts are treated as failures. The `responseTime` for a timeout will equal the `timeout_ms` value.

---

## Ideas

- [ ] Response body validation: check that the response body contains a specific string or JSON field
- [ ] Latency alerts: alert when response time exceeds a threshold even if the status is correct
- [ ] Weekly uptime report: aggregate the Incidents tab into a weekly summary email
- [ ] Status page: serve a simple public status page from the Endpoints sheet data

---

## License

MIT.
