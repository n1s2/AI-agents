# api-rate-limit-planner

A single global rate limit applied to every endpoint either blocks legitimate bursty usage on cheap endpoints or lets abusive traffic through on expensive ones. This designs a rate limiting strategy calibrated per endpoint by cost and criticality, with tier-based limits for different customer types, proper response headers, graceful degradation before hard blocks, and abuse detection signals distinct from legitimate high usage.

---

## What it does

Takes API name, infra capacity, usage patterns per endpoint (average/peak requests per minute, request cost, client type, criticality), customer tiers, and abuse prevention goals. Claude produces:

- **Strategy summary** and **algorithm recommendation** — token_bucket/sliding_window/fixed_window/leaky_bucket with rationale
- **Endpoint limits** — specific limit and window per endpoint, burst allowance, rationale
- **Tier-based limits** — overall limits per customer tier with rationale
- **Response headers** — specific rate limit headers to return (e.g., X-RateLimit-Remaining)
- **Exceeded behavior** — HTTP status, example response body, retry-after guidance
- **Graceful degradation** — warnings before hard blocking as clients approach limits
- **Abuse detection signals** — patterns that indicate abuse versus legitimate high usage
- **Monitoring metrics** — what to track for rate limit effectiveness
- **Documentation recommendations** — what to tell API consumers

HTML report with tier cards, endpoint limit table, response headers as code blocks, and example 429 response body.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/plan-rate-limits \
  -H "Content-Type: application/json" \
  -d '{
    "api_name": "Flowdesk Public API v2",
    "infra_capacity": "Backend can sustain 5000 req/sec across all endpoints before database connection pool saturation",
    "abuse_prevention_goals": "Prevent scraping of task data, prevent brute-force on auth endpoints, allow legitimate integrations to sync freely",
    "fairness_goals": "Free tier should not be able to degrade service for paid tiers",
    "reply_email": "platform@flowdesk.com",
    "customer_tiers": [
      {"tier": "free", "typical_usage": "low, occasional API access"},
      {"tier": "business", "typical_usage": "moderate, regular sync integrations"},
      {"tier": "enterprise", "typical_usage": "high volume, real-time sync needs"}
    ],
    "usage_patterns": [
      {"endpoint": "GET /api/v2/tasks", "avg_requests_per_min": 12, "peak_requests_per_min": 180, "request_cost": "low", "client_type": "mixed", "criticality": "high"},
      {"endpoint": "POST /api/v2/tasks/bulk-assign", "avg_requests_per_min": 2, "peak_requests_per_min": 15, "request_cost": "high", "client_type": "external", "criticality": "medium"},
      {"endpoint": "POST /api/v2/auth/login", "avg_requests_per_min": 5, "peak_requests_per_min": 40, "request_cost": "medium", "client_type": "external", "criticality": "critical"},
      {"endpoint": "GET /api/v2/export", "avg_requests_per_min": 0.5, "peak_requests_per_min": 5, "request_cost": "high", "client_type": "external", "criticality": "low"}
    ]
  }'
```

**Required:** `api_name`, `usage_patterns`

---

## Algorithm selection

- **token_bucket** — good for bursty legitimate traffic (allows short bursts above the steady rate)
- **sliding_window** — smoother, prevents edge-of-window burst exploitation of fixed windows
- **fixed_window** — simplest to implement, but vulnerable to burst-at-boundary abuse
- **leaky_bucket** — smooths traffic to a constant outflow rate, good for protecting downstream systems with fixed capacity

Claude picks based on the usage patterns and criticality mix you describe.

---

## Cost-calibrated limits

Endpoints are limited based on their actual cost to your infrastructure, not a uniform number. A cheap read endpoint (`GET /tasks`) gets a much higher limit than an expensive bulk operation or export endpoint, reflected in the `request_cost` field per pattern.

---

## Graceful degradation

Rather than a hard cliff at the limit, Claude designs a warning zone — clients see remaining-quota headers dropping and can slow down before hitting a hard 429, rather than being surprised by sudden failures.

---

## Limitations

- Recommendations are based on the usage patterns and infra capacity you describe — actual limits should be validated with load testing before production deployment.
- This designs the strategy; implementing it requires your actual rate limiting infrastructure (Redis-based limiter, API gateway config, etc).

---

## License

MIT.
