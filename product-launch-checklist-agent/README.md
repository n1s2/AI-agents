# product-launch-checklist-agent

Product launches fail in predictable ways. The legal review didn't happen. Nobody updated the status page. The support team wasn't briefed. Analytics weren't set up. The on-call rotation wasn't covered for launch weekend. These aren't random failures — they're the things that consistently get skipped when teams are heads-down on the product itself.

This generates a complete, phase-organized launch checklist calibrated to your product type, timeline, channels, and team size. Tasks are assigned to owners, blockers are flagged explicitly, and each phase has a realistic time window. It also generates a launch-day runbook with contingency plans, go/no-go criteria, a post-launch monitoring plan, and a specific list of things teams typically overlook for your combination of product type and launch channels.

---

## What it does

1. Accepts a POST: product name, product type, launch date, launch channels, team size, existing audience, target market, key features, launch goal, known risks
2. Calculates days until launch
3. Claude generates:
   - Launch readiness assessment (honest about tight timelines)
   - Critical path items (what will actually block the launch)
   - Phased checklist: T-30 to T-14, T-14 to T-7, T-7 to T-1, launch day, T+7 — each with tasks, owners, blocker flags, time estimates, notes
   - Launch day runbook: hour-by-hour with contingency plans per step
   - Go/no-go criteria (verifiable conditions to check before going live)
   - Post-launch monitoring: metrics, alert thresholds, day-1 and week-1 tasks
   - Commonly overlooked items for this specific product type and channel
4. Builds an HTML checklist with visual phase cards and checkbox items
5. Emails if `reply_email` provided
6. Returns full JSON

---

## Stack

- **n8n** — webhook + workflow
- **Anthropic Claude** (claude-sonnet-4-20250514) — checklist generation
- **SMTP** — optional email delivery

---

## Setup

### 1. Environment variables

```
FROM_EMAIL=launches@yourcompany.com
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** (optional)

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/launch-checklist \
  -H "Content-Type: application/json" \
  -d '{
    "product_name": "Flowdesk 2.0",
    "product_type": "saas",
    "launch_date": "2025-06-15",
    "launch_channels": ["product_hunt", "email_list", "twitter", "linkedin", "press"],
    "team_size": 8,
    "existing_audience": true,
    "target_market": "Small business owners and ops managers at 5-20 person companies",
    "key_features": "New AI-powered task prioritization, improved mobile app, Slack integration, bulk import from Asana/Trello",
    "launch_goal": "500 signups in first week, 50 paid conversions within 30 days",
    "known_risks": "Slack integration has had some instability in staging. Mobile app not yet submitted to App Store — need approval timeline.",
    "previous_launches": "v1.0 launched 18 months ago via Product Hunt — got 800 upvotes but poor email conversion",
    "reply_email": "pm@flowdesk.com"
  }'
```

**Required:** `product_name`, `launch_date`, `product_type`

---

## Product types

| Type | Calibration |
|---|---|
| `saas` | Status page, pricing page, trial flow, onboarding email sequence |
| `mobile_app` | App Store/Play Store submission timing, ASO, push notification setup |
| `physical_product` | Inventory, shipping, packaging, fulfillment, returns |
| `digital_download` | Payment flow, delivery email, license key management |
| `marketplace_listing` | Platform-specific requirements, review solicitation |
| `api_product` | Documentation, SDKs, rate limit communication, status page |
| `content_product` | DRM, platform exclusivity, affiliate setup |
| `service_launch` | Capacity planning, intake form, SLA communication |
| `feature_release` | Feature flags, rollout strategy, customer communication |

The checklist structure and commonly-overlooked items adapt significantly by type.

---

## The critical path

The `critical_path_items` section identifies 3–5 things most likely to block or derail the launch. For a SaaS with an App Store submission in flight, that's the App Store review timeline. For a Product Hunt launch, that's scheduling the post and having a hunter lined up. For a physical product, it's fulfillment lead times.

These get called out explicitly rather than buried in the checklist.

---

## Launch day runbook

The runbook is hour-by-hour for launch day with:
- **Task** — what to do
- **Owner** — who does it
- **If it fails** — the contingency plan if that step doesn't work as expected

This is the document you pull up at 6am on launch day. It shouldn't require any decisions — just execution.

---

## Go/no-go criteria

Five to seven specific, verifiable conditions that must be true before going live. Not vague ("product is ready") but concrete ("staging environment has had zero critical errors in the last 24 hours", "support team has completed the product briefing", "pricing page has been reviewed by legal"). If any are false on launch morning, the launch waits.

---

## Post-launch monitoring

The monitoring section gives:
- **Metrics to watch** — what to check in the first 48 hours (signup rate, error rate, support ticket volume, etc.)
- **Alert thresholds** — specific numbers that should trigger action ("if error rate exceeds 2%, page the on-call engineer")
- **Day-1 tasks** — what the team does the day after launch
- **Week-1 tasks** — the first week follow-up (press follow-up, user interviews, performance analysis)

---

## Commonly overlooked

This section is calibrated to your specific product type and channel combination. For a SaaS launching on Product Hunt:
- "Product Hunt posts can only be made between midnight and 12:01 AM Pacific — schedule exactly"
- "Analytics tracking on the PH referral URL often breaks — test your UTM parameters"
- "Support ticket volume typically 3–5x normal on launch day — staff accordingly"

For a mobile app:
- "App Store review takes 1–3 days but can be longer — submit 2 weeks before target date"
- "Push notification permissions prompt timing affects opt-in rate significantly"

---

## Timeline flexibility

The webhook computes `daysUntilLaunch` from today's date. Claude uses this to assess whether the timeline is realistic and calibrates what can actually be done in the remaining time. If `daysUntilLaunch` is 7, Claude won't generate 6 weeks of pre-launch tasks — it focuses on what's achievable.

---

## Limitations

- The checklist is comprehensive but not exhaustive for every edge case. Legal/regulatory requirements (FDA approval for medical products, FCC certification for hardware, financial services licensing) require specialist input beyond what this agent produces.
- Owner assignment is based on common team structures. If your team names roles differently (e.g. "growth" instead of "marketing"), update the labels in the prompt.
- The checklist is generated once. It doesn't track completion or send reminders as items come due. To add tracking, pipe the task list into a project management tool (Notion, Linear, Asana) via their APIs.

---

## Ideas

- [ ] Notion integration: auto-create a Notion database with each task as a page, assigned to the right person
- [ ] Countdown reminder: daily Slack post counting down with that day's due tasks
- [ ] Post-launch retrospective: a companion webhook that generates a launch retrospective template 2 weeks post-launch
- [ ] Template library: save generated checklists and tag by product type for reuse on future launches

---

## License

MIT.
