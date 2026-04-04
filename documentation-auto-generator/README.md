# FLOOWBOX - Documentation Auto-Generator

Documentation is always out of date because nobody writes it when pushing code. This workflow generates documentation automatically on every push — and commits it back to the repo.

## What it does

Triggers on every GitHub push event. Scans the changed files for code files (JavaScript, TypeScript, Python, Go, Java, PHP, Ruby). For each code file, fetches the content via GitHub API and sends it to GPT-4o which generates comprehensive documentation — file overview, per-function docs with parameters and return types, usage examples, and dependency explanations. The docs get committed as Markdown files to the `/docs/` folder automatically. Also saves to a Notion wiki page and notifies Slack.

## Tools Used
- **Orchestration:** n8n
- **Trigger:** GitHub Push webhook
- **Code Fetching:** GitHub Contents API
- **Documentation:** OpenAI GPT-4o
- **Output:** GitHub API (commit Markdown to /docs/)
- **Wiki:** Notion
- **Notification:** Slack

## Flow

```
GitHub push → webhook fires
  → Find changed code files (skip tests/specs)
  → For each file (max 5 per push):
      → Fetch content via GitHub API
      → Decode base64 content
      → GPT-4o: generate full documentation
      → Commit Markdown to /docs/filename.md
      → Save to Notion wiki
  → Slack: docs generated notification
```

## Generated documentation structure

```markdown
# auth.js

Handles user authentication and JWT token management for the API.

## Functions

### `generateToken(userId, role)`

Generates a signed JWT token for authenticated users.

**Parameters:**
- `userId` (string): The unique user identifier
- `role` (string): User role — 'admin', 'user', or 'viewer'

**Returns:** `string` — Signed JWT token valid for 24 hours

**Example:**
```javascript
const token = generateToken('user_123', 'admin');
// Returns: 'eyJhbGci...'
```

**Notes:** Tokens expire after 24 hours. Use `refreshToken()` for renewal.
```

## Why I built this

A client's codebase had zero documentation — functions named `processData()` and `handleThing()`. Onboarding new developers took weeks. This runs silently in the background and builds up a documentation layer automatically as the team codes normally.

## Supported languages

JavaScript, TypeScript, Python, Go, Java, PHP, Ruby

## Setup

1. GitHub Personal Access Token (repo read + write)
2. OpenAI API key
3. Notion integration + Docs Database ID
4. Slack Bot Token + #engineering channel
5. GitHub webhook: Push events
