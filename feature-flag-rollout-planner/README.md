# feature-flag-rollout-planner

Flipping a feature flag to 100% the day it's code-complete is how minor bugs become major incidents. A proper rollout uses progressive exposure with clear advance/abort criteria decided in advance — not judgment calls made in the panic of a live incident. This generates a staged rollout plan calibrated to risk level, with specific success and abort criteria per stage, a tested rollback plan, and a communication plan for each audience.

---

## What it does

Takes feature name, description, risk profile, user base, technical risk, rollback complexity, and available segmentation options. Claude produces:

- **Rollout summary** — overall approach and why
- **Stages** — each with: specific audience (% or segment), duration before proceeding, success criteria to advance, abort criteria to roll back immediately, metrics to monitor, and who to notify
- **Technical checklist** — pre-rollout items to verify
- **Rollback plan** — trigger conditions, actual rollback steps, time estimate, and data cleanup needed if rolled back after partial use
- **Communication plan** — per audience (internal_team/support/sales/customers) with when and what message
- **Kill switch readiness** — specific verification that instant-disable capability exists and is tested
- **Post-rollout cleanup** — steps once fully rolled out (removing flag, removing old code path)

HTML report with color-coded stage cards showing advance/abort criteria side by side, and a full communication plan.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/plan-feature-rollout \
  -H "Content-Type: application/json" \
  -d '{
    "feature_name": "Bulk task assignment",
    "feature_description": "Allows users to select multiple tasks and assign them to a team member in one action. New UI component, new API endpoint, touches the core task assignment path used by all users.",
    "risk_profile": "medium",
    "user_base": "12,000 active weekly users across 400 workspaces",
    "technical_risk": "New database write pattern doing up to 100 row updates in one transaction. Could add load to the tasks table during peak hours if usage is high.",
    "rollback_complexity": "simple",
    "target_timeline": "2 weeks from internal testing to full rollout",
    "dependencies_or_prereqs": "Requires the new bulk-assign API endpoint to be deployed and load tested first",
    "segmentation_options": ["percentage rollout", "workspace_id allowlist", "plan tier"],
    "business_metrics_to_watch": ["task assignment error rate", "API p95 latency", "support ticket volume"],
    "reply_email": "sara@flowdesk.com"
  }'
```

**Required:** `feature_name`, `feature_description`

---

## Stage progression

Typical progression: internal team → small % of low-risk segment → larger % → full rollout. The exact stages and percentages are calibrated to risk profile — a high-risk feature gets more, smaller stages; a low-risk feature might go from internal testing straight to 50%.

---

## Advance/abort criteria decided upfront

The most important part of this plan: success and abort criteria for each stage are specific and measurable, decided before the rollout starts. "Error rate stays below 0.5%" or "no increase in P95 latency" — not vague judgment calls made under pressure when something looks slightly off during a live rollout.

---

## Kill switch readiness

Claude explicitly checks that you have a tested way to instantly disable the feature — not just "we'll remove the flag eventually" but a verified, fast mechanism. This is often the difference between a 2-minute incident and a 2-hour one.

---

## Limitations

- The plan is a template calibrated to the risk and context you provide — actual stage percentages and durations should be adjusted based on your specific traffic patterns and monitoring capabilities.
- This plans the rollout — it doesn't configure your actual feature flag system (LaunchDarkly, Split, custom). Use the plan to configure your tooling.

---

## License

MIT.
