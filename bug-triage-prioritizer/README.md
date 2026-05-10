# bug-triage-prioritizer

Bug reports come in at all hours from engineers, QA, customers, and support. Without triage, the queue fills with a mix of critical regressions and minor display glitches, all sitting at the same priority level, and the team wastes time deciding what to work on instead of actually working on it.

This is two things. A webhook that accepts a bug report, runs it through Claude which assesses the real severity (not what the reporter thinks it is), identifies the probable cause, suggests the owning team, and gives the first concrete debugging step. Critical bugs get an immediate email to the engineering lead. Everything logs to Google Sheets.

The second part is a daily 9am digest — loads the full open bug queue, flags anything that's gone stale past its severity SLA, and emails a summary with Claude's read on queue health and what needs action today.

---

## What it does

**Bug triage (webhook, `/triage-bug`):**
- Accepts: title, description, steps to reproduce, expected/actual behavior, environment, affected component, reported severity
- Claude assesses: severity (may differ from reporter's), bug type, reproducibility, affected scope, probable cause, suggested owner, first debug step, workaround existence, clarification needed
- Saves to Google Sheets
- Critical bugs → immediate email to engineering lead
- Optional Slack notification per bug
- Returns full triage JSON

**Daily digest (every day 9am):**
- Loads all open bugs from Google Sheets
- Flags stale bugs (critical 3+ days, high 7+ days, medium 14+ days)
- Claude writes a queue health note
- Emails formatted digest with bug table to engineering lead

---

## Stack

- **n8n** — webhook + daily scheduler
- **Google Sheets** — bug log
- **Anthropic Claude** (claude-sonnet-4-20250514) — triage + queue analysis
- **SMTP** — email alerts + daily digest
- **Slack** (optional) — per-bug notifications

---

## Setup

### 1. Create the Google Sheet

One tab: **Bugs** — columns:
```
bug_id | reported_at | title | reporter | environment | severity | bug_type | reproducibility | suggested_owner | workaround_exists | needs_clarification | status | description
```

### 2. Environment variables

```
BUGS_SHEET_ID=your_google_sheet_id
FROM_EMAIL=bugs@yourcompany.com
ENG_LEAD_EMAIL=lead@yourcompany.com
```

### 3. Credentials

- **Google Sheets OAuth2**
- **Anthropic API** (LangChain node)
- **SMTP**
- **Slack API** (optional)

### 4. Import and activate

Import `workflow.json`, activate. Two webhook URLs will be visible — use the one for `/triage-bug`. The daily digest runs automatically at 9am.

---

## Submitting a bug

```bash
curl -X POST https://your-n8n.com/webhook/triage-bug \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Checkout button unresponsive after applying discount code",
    "description": "After entering a valid discount code on the cart page, the Apply button shows a spinner briefly then returns to normal, but the Checkout button becomes unresponsive — clicks do nothing. Refreshing the page loses the cart contents.",
    "steps_to_reproduce": "1. Add any item to cart\n2. Go to cart page\n3. Enter discount code SAVE20\n4. Click Apply\n5. Click Checkout — nothing happens",
    "expected_behavior": "Checkout proceeds to payment page",
    "actual_behavior": "Checkout button click has no effect, no error shown",
    "reporter": "Priya Nair",
    "reporter_email": "priya@company.com",
    "affected_component": "checkout",
    "affected_users": "anyone using a discount code — estimated 15-20% of orders",
    "environment": "production",
    "reported_severity": "critical",
    "browser_or_platform": "Chrome 124, macOS",
    "error_message": "Uncaught TypeError: Cannot read properties of null (reading dispatch) at checkout.js:847",
    "notify_slack_channel": "#eng-bugs"
  }'
```

**Required:** `title`, `description`, `reporter`

---

## Example triage response

```json
{
  "bug_id": "BUG-1746432891234",
  "severity": "critical",
  "severity_rationale": "Blocks checkout for ~15-20% of orders — direct revenue impact with no visible error message to the user.",
  "severity_matches_reporter": true,
  "bug_type": "logic_error",
  "reproducibility": "always",
  "probable_cause": "The discount code dispatch handler nullifies a checkout state reference (line 847 in checkout.js) — likely a race condition or incorrect null check introduced in a recent deploy.",
  "suggested_owner": "frontend",
  "first_debug_step": "Check git blame on checkout.js:847 for recent changes and compare the discount code application flow in staging vs production.",
  "workaround_exists": true,
  "workaround_description": "Remove the discount code from the cart, proceed to checkout, then contact support to apply the discount manually post-purchase.",
  "needs_clarification": false,
  "clarification_questions": [],
  "summary_for_ticket": "Checkout button becomes unresponsive after applying a discount code. JS error at checkout.js:847 suggests a null reference in the dispatch handler. Affects ~15-20% of orders. Introduced in production — likely a recent deploy regression."
}
```

---

## Severity override

Claude independently assesses severity regardless of what the reporter submitted. If a reporter marks something critical but it's actually a display issue affecting 0.1% of users with a workaround, Claude will set it to medium and explain why in `severity_mismatch_note`.

This is intentional. Reporters — especially non-technical ones — tend to over-escalate. The triage should reflect actual impact, not reported urgency.

If you disagree with the triage, update the `severity` column in the sheet directly.

---

## Stale bug SLAs

The daily digest flags bugs that have exceeded these age thresholds without being closed:
- **Critical:** 3 days
- **High:** 7 days
- **Medium:** 14 days
- **Low:** no SLA (shown but not flagged)

Adjust the thresholds in the **Analyse Bug Queue** node's JavaScript.

---

## Closing bugs

When a bug is resolved, update the `status` column in the sheet to `closed`. The daily digest filters to non-closed bugs, so it won't appear in future reports.

Build a companion `/close-bug` webhook if you want to close programmatically — it just needs to find the row by `bug_id` and update `status`.

---

## Integrating with issue trackers

The triage response includes `summary_for_ticket` — a clean 2-3 sentence description ready to paste into Jira, Linear, or GitHub Issues. To auto-create tickets, add a step after **Save to Bug Sheet** that calls the Linear or Jira API with the triage data.

---

## Limitations

- Claude triages based on the description quality. Vague reports ("it's broken") produce vague triage. The more detail in `description`, `steps_to_reproduce`, and `error_message`, the better the output.
- `affected_users` is a free-text estimate from the reporter — Claude uses it to inform severity but can't independently verify it.
- The daily digest shows up to 20 bugs in the table. If your queue is larger, adjust the `.slice(0, 20)` in the **Build Digest Email** node.

---

## Ideas

- [ ] Auto-assign: map `suggested_owner` to a real team member in a lookup table and assign directly in Linear/Jira
- [ ] Duplicate detector: before saving, check if a similar bug already exists in the sheet
- [ ] Reporter acknowledgement: if `reporter_email` is provided, send an automated "we've received your report" email
- [ ] Weekly trends: compare this week's bug counts to last week, surface regression patterns

---

## License

MIT.
