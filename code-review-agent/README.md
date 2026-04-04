# FLOOWBOX - Code Review Agent (GitHub PR)

Every pull request gets an AI code review within seconds of being opened — before any human reviewer even sees it. Catches bugs, security issues, and style problems automatically.

## What it does

Listens for GitHub PR webhook events. When a PR is opened, fetches the full diff automatically. GPT-4o reviews the code for bugs, logic errors, security vulnerabilities, performance issues, missing error handling, and test coverage gaps. Posts the complete review directly on the PR via GitHub API with an APPROVE or REQUEST_CHANGES verdict. Notifies Slack with a summary.

## Tools Used
- **Orchestration:** n8n
- **Trigger:** GitHub webhook (PR opened events)
- **Diff Fetching:** GitHub API
- **Review Engine:** OpenAI GPT-4o
- **Output:** GitHub PR Review API
- **Notification:** Slack

## Flow

```
GitHub PR opened → webhook fires
  → Filter: only 'opened' events
  → Extract: PR number, title, diff URL, repo, author
  → Fetch full diff from GitHub
  → Truncate to 8000 chars if needed
  → GPT-4o: comprehensive code review
  → Post review comment on PR (GitHub API)
  → Slack: verdict summary
```

## What GPT-4o reviews

```json
{
  "overall_verdict": "request_changes",
  "critical_issues": [
    {"file": "auth.js", "line": "47", "issue": "SQL query uses string concatenation", "suggestion": "Use parameterized queries to prevent SQL injection"}
  ],
  "security_issues": ["Hardcoded API key in config.js line 12"],
  "performance_issues": ["N+1 query in UserController.getAll()"],
  "missing_tests": ["No test for error case in payment handler"],
  "praise": ["Clean separation of concerns in service layer"]
}
```

## GitHub setup

1. Go to your repo → Settings → Webhooks → Add webhook
2. Set Payload URL to your n8n webhook URL
3. Content type: `application/json`
4. Events: select "Pull requests"

## Why I built this

Small engineering teams often skip code review because it takes time. This ensures every PR gets at least one review pass instantly — catching obvious issues before a human reviewer spends time on them. The AI catches security and performance issues that are easy to miss under time pressure.

## Setup

1. GitHub Personal Access Token (repo permissions)
2. OpenAI API key
3. Slack Bot Token + #engineering channel
4. Configure GitHub webhook in your repo
