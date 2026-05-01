# github-pr-review-digest

Code review velocity is one of those metrics that sounds boring until you realize your team has 8 open PRs, two of them are 5 days old with no reviewers, and nobody noticed because everyone assumed someone else was handling it.

This runs hourly, pulls all open PRs from a GitHub repo, classifies them (stale, needs reviewer, recently updated, draft), sends the data to Claude which writes an opinionated digest about the queue's health, and delivers it to the team as an email and optionally a Slack message.

The point isn't to add another meeting or nag people — it's to surface what's getting stuck so it doesn't silently rot.

---

## What it does

- Runs every hour
- Fetches all open PRs from a GitHub repo via the REST API
- Classifies each PR:
  - **Stale**: 3+ days old with no activity
  - **Needs reviewer**: open 4+ hours, no reviewer assigned
  - **Recently updated**: changed in the last hour
  - **Draft**: not ready for review
- Sends the full picture to Claude, which writes a short opinionated digest: overall queue health, what needs action, any risky or notable PRs, a concrete team suggestion
- Formats it into a GitHub-themed HTML email with a PR table (color-coded by age and size)
- Sends to a team email list
- Optionally posts to a Slack channel

---

## Stack

- **n8n** — scheduling + workflow
- **GitHub REST API** — PR data
- **Anthropic Claude** (claude-opus-4-5) — queue analysis
- **SMTP** — email delivery
- **Slack** (optional) — channel digest

---

## Setup

### 1. GitHub token

Create a classic personal access token at [github.com/settings/tokens](https://github.com/settings/tokens) with `repo` scope (read-only is sufficient for this workflow). If your org uses fine-grained tokens, you need `Pull requests: Read` permission on the target repo.

### 2. Environment variables

```
GITHUB_OWNER=your-org-or-username
GITHUB_REPO=your-repo-name
GITHUB_TOKEN=ghp_your_token_here
FROM_EMAIL=digest@yourcompany.com
TEAM_EMAIL=engineering@yourcompany.com
SLACK_CHANNEL=#pr-digest          # leave empty to disable Slack
```

### 3. Credentials

- **Anthropic API** — LangChain node
- **SMTP** — your mail provider
- **Slack API** — only if `SLACK_CHANNEL` is set

### 4. Import and activate

Import `workflow.json`, set your env vars, activate. It runs every hour but you can trigger manually to test.

---

## Frequency

Hourly is aggressive for some teams. If your team is smaller or doesn't move that fast:

In the **Every Hour** trigger node, change `hoursInterval` to `4` for 4x daily, or switch the field to `days` with `triggerAtHour: 9` for a single daily digest.

A good middle ground for most teams: run hourly during business hours only. You'd need to add a time-window check in the **Classify PRs** node — doable but not built in by default.

---

## Multiple repos

The workflow is wired for one repo. For multiple repos:
1. Store repo list in a Google Sheet or env var (comma-separated)
2. Add a Split In Batches node before the GitHub fetch
3. Loop through each repo

I haven't built this yet because I only needed one repo. If you need multi-repo, the structure is straightforward to extend.

---

## What Claude looks at

Claude gets:
- Every open PR: number, title, author, age, files changed, line diff, reviewers, labels
- Counts of stale, urgent, draft PRs
- The raw data to form opinions about queue health

It's told to write like a teammate, not a bot. The output varies based on what's actually in the queue — a healthy queue with 2 small PRs and active reviewers gets a different (shorter) analysis than a queue with 9 PRs and three unreviewed for 4 days.

---

## PR size color coding

In the email table, the +additions/-deletions cell is color-coded:
- 🟢 Green: under 200 lines total
- 🟡 Amber: 200–500 lines
- 🔴 Red: 500+ lines

This is a rough signal. A 600-line PR that's all test files is different from a 600-line core refactor. Claude's analysis picks up on context when it's visible in the PR title.

---

## Known limitations

- Only covers one repo per workflow instance
- Draft PRs are tracked but Claude is told to focus on review-ready PRs — drafts appear in the table but rarely get highlighted in the analysis unless they've been sitting for a very long time
- GitHub API rate limit is 5,000 requests/hour for authenticated requests. Running this hourly for one repo uses 1 request per run — not a concern unless you're running many copies
- The `additions` and `deletions` fields from the GitHub list PRs endpoint aren't always populated for large repos — you may see 0/0 for some PRs

---

## Things I want to add

- [ ] Per-author stats (who has the most PRs waiting for their review)
- [ ] Weekly summary of average time-to-merge
- [ ] Auto-comment on stale PRs with a gentle ping to the author
- [ ] Filter by label (e.g. only digest PRs labeled `ready-for-review`)

---

## License

MIT.
