# daily-standup-writer

Async standups are better than synchronous ones for most teams. The problem is nobody wants to write them. By 9am you're already deep in something, and stopping to compose a coherent update feels like overhead.

This takes a brain-dump — bullet points, fragments, stream of consciousness — and turns it into a clean standup in whatever format your team uses. It posts to Slack if you want, logs to a sheet for team history, and at 9:15am posts a digest of everyone who submitted that morning.

---

## What it does

**Individual standup (webhook):**
- POST raw notes, name, team, optional blockers
- Claude formats into clean standup: yesterday/today/blockers, async paragraph, or bullets
- Saves to Google Sheets
- If `slack_channel` provided: posts directly
- Returns formatted standup as JSON

**Team digest (daily 9:15am):**
- Loads all standups logged today from the sheet
- Posts full team digest to `DIGEST_SLACK_CHANNEL`
- Silent if nobody has submitted

---

## Stack

- **n8n** — webhook + daily scheduler
- **Google Sheets** — standup log
- **Anthropic Claude** (claude-sonnet-4-20250514) — formatting
- **Slack** — individual post + team digest

---

## Setup

### 1. Create the Standups sheet

Tab: **Standups** — columns:
```
date | team_member | team | raw_update | formatted_standup | mood | logged_at
```

### 2. Environment variables

```
STANDUP_SHEET_ID=your_google_sheet_id
DIGEST_SLACK_CHANNEL=#standup
```

### 3. Credentials

- **Google Sheets OAuth2**
- **Anthropic API** (LangChain node)
- **Slack API**

### 4. Import and activate

Import `workflow.json`, activate. Digest fires at 9:15am. Individual standups go through the webhook anytime.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/standup \
  -H "Content-Type: application/json" \
  -d '{
    "team_member_name": "Rahul Mehta",
    "team": "Backend",
    "raw_update": "yesterday finished auth refactor, took longer due to oauth edge cases. today working on rate limiter, should be done eod. need to review priya pr. staging env has been flaky",
    "blockers": "staging environment instability",
    "mood": 4,
    "standup_format": "yesterday_today_blockers",
    "max_words": 80,
    "slack_channel": "#backend-standup",
    "date": "2025-05-07"
  }'
```

**Required:** `team_member_name`, `raw_update`

---

## Formats

| Format | Style |
|---|---|
| `yesterday_today_blockers` | Three sections with headers and bullets (default) |
| `async_message` | Single paragraph, no headers, natural Slack message |
| `bullet_only` | Flat bullet list, no section headers |

---

## What Claude does and doesn't do

Claude formats — it doesn't invent. If you wrote "finished auth thing, working on rate limiter", that's what goes in. It won't add context you didn't provide or flesh out vague items.

It removes filler, structures the sections properly, and cleans up grammar. Your words, just organized.

---

## The team digest

At 9:15am the digest loads everyone who submitted today and posts the combined update to `DIGEST_SLACK_CHANNEL`. The async standup meeting replacement — everyone reads the digest instead of joining a Zoom call.

If nobody has submitted by 9:15am, no message is posted.

---

## Limitations

- Claude keeps your words intact. Incoherent input produces slightly-cleaner incoherent output — it won't hallucinate clarity that wasn't there.
- The digest time is hardcoded. Change `triggerAtHour` and `triggerAtMinute` in the scheduler node to adjust.
- No deduplication — if someone submits twice in one day, both appear in the digest.

---

## Ideas

- [ ] Slack slash command: `/standup [your notes]` posts directly
- [ ] Weekly rollup: Friday summary of the week's standups per team member
- [ ] Blocker tracker: extract blockers, track resolution time
- [ ] Mood trend: weekly mood score chart for team health monitoring

---

## License

MIT.
