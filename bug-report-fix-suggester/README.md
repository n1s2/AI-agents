# FLOOWBOX - Bug Report to Fix Suggester Agent

When a bug is reported, developers want two things immediately: what caused it and how to fix it. This agent delivers both automatically the moment a bug issue is opened on GitHub.

## What it does

Triggers on GitHub issues labeled "bug". Searches the codebase for files related to the bug title. GPT-4o analyzes the bug description plus relevant code context to identify the root cause and generate an exact fix with a code snippet. Posts the analysis as a comment on the GitHub issue. Labels the issue by severity. Critical bugs fire an urgent Slack alert to the engineering channel.

## Tools Used
- **Orchestration:** n8n
- **Trigger:** GitHub Issues webhook (labeled "bug")
- **Code Search:** GitHub Search API
- **Analysis:** OpenAI GPT-4o
- **Output:** GitHub Issues API (comment + label)
- **Alerts:** Slack (critical bugs)

## Flow

```
GitHub issue opened with "bug" label
  → Extract: title, body, repo
  → GitHub Search API: find related code files
  → GPT-4o: root cause + fix suggestion
  → Post comment on issue (GitHub API)
  → Add severity label
  → IF critical: Slack alert
```

## What gets posted on the issue

```markdown
## AI Bug Analysis

**Root Cause:** The `getUserById` function does not handle the case where 
`user_id` is null, causing an uncaught TypeError in the authentication middleware.

**Fix:**
```javascript
// Before
const user = await User.findById(userId);

// After  
if (!userId) return res.status(400).json({ error: 'User ID required' });
const user = await User.findById(userId);
```

**Files to change:** `middleware/auth.js` (line 47)
**Test steps:** 1. Send request without user_id header 2. Expect 400 response
**Estimated fix time:** 30 minutes
```

## Why I built this

Engineering teams lose hours triaging bugs — reproducing, finding the relevant code, figuring out root cause. This compresses the triage step from hours to seconds. The developer still writes the actual fix, but they start with a clear analysis rather than a blank screen.

## Setup

1. GitHub Personal Access Token
2. OpenAI API key
3. Slack Bot Token + #engineering channel
4. GitHub webhook: Issues events, label filter for "bug"
