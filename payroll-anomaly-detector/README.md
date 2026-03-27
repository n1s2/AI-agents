# FLOOWBOX - Payroll Anomaly Detector

Payroll errors are expensive and embarrassing. This workflow audits every payroll run before it's processed — catching salary spikes, missing employees, and duplicate entries automatically.

## What it does

Runs on the 25th of every month. Pulls current and previous month payroll data from Google Sheets and compares them. GPT-4o acts as an AI payroll auditor — flagging salary changes over 20%, employees appearing or disappearing, duplicate entries, and total payroll variance over 15%. High-severity issues trigger an urgent Slack alert. A full report always goes to the finance email.

## Tools Used
- **Orchestration:** n8n
- **Data:** Google Sheets (current + previous month payroll)
- **AI Auditor:** OpenAI GPT-4o
- **Alerts:** Slack (urgent channel)
- **Report:** Email to finance team
- **Schedule:** Monthly on the 25th

## Flow

```
25th of month, 10 AM
  → Fetch current month payroll (Google Sheets)
  → Fetch previous month payroll (Google Sheets)
  → Merge both datasets
  → GPT-4o compares and flags anomalies
  → Parse: severity classification
  → IF high severity: urgent Slack alert
  → ELSE: normal Slack digest
  → Always: email full report to finance
```

## What GPT-4o detects

```json
{
  "total_current": 850000,
  "total_previous": 720000,
  "variance_percent": 18,
  "anomalies": [
    {"employee": "Rahul Sharma", "type": "salary_spike", "detail": "Salary increased 45% vs last month", "severity": "high"},
    {"employee": "Priya Mehta", "type": "missing", "detail": "Present last month, not in current payroll", "severity": "high"}
  ],
  "new_employees": ["Ankit Singh"],
  "removed_employees": [],
  "summary": "18% variance detected with 2 high-severity anomalies requiring review before processing."
}
```

## Why I built this

A finance client processed a payroll with a data entry error — one employee's salary was entered as ₹850,000 instead of ₹85,000. Caught the next month after the money was already sent. This catches it before processing.

## Google Sheets schema

| Column | Description |
|---|---|
| Employee Name | Full name |
| Employee ID | Unique ID |
| Department | Team/dept |
| Basic Salary | Monthly base |
| Allowances | Total allowances |
| Deductions | PF, tax, etc. |
| Net Pay | Final amount |

## Setup

1. Google Sheets with current + previous month tabs
2. OpenAI API key
3. Slack Bot Token + #payroll-alerts channel
4. SMTP credentials for finance email
