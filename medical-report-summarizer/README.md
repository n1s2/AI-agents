# FLOOWBOX - Medical Report Summarizer Agent

Lab reports and diagnostic documents are written for doctors, not patients. This workflow converts any medical report into plain language — explaining what each finding means, what needs attention, and what to do next.

## What it does

Accepts a medical report URL (lab results, radiology report, discharge summary, etc.). Jina AI fetches the full text. GPT-4o extracts all test results with their values, normal ranges, and status — normal, high, low, or critical. Critical values trigger an immediate Slack alert. A second pass writes a plain-language summary tailored to the specified audience (patient, family, nurse) explaining what each abnormal finding means practically and what action to take.

## Tools Used
- **Orchestration:** n8n
- **Content Extraction:** Jina AI
- **Medical Analysis:** OpenAI GPT-4o (Pass 1 — data extraction)
- **Plain Language:** OpenAI GPT-4o (Pass 2 — patient summary)
- **Urgent Alerts:** Slack
- **Trigger:** Webhook

## Flow

```
POST: {report_url, type, patient_context, audience}
  → Jina AI: fetch report text
  → GPT-4o: extract all test values + flag abnormals
  → IF critical values: Slack urgent alert
  → GPT-4o: write plain language summary for audience
  → Return: summary + needs attention + next steps
```

## Test result extraction

```json
{
  "test_results": [
    {"test_name": "HbA1c", "value": "8.2%", "normal_range": "Below 5.7%", "status": "high"},
    {"test_name": "Hemoglobin", "value": "13.5 g/dL", "normal_range": "13-17 g/dL", "status": "normal"}
  ],
  "abnormal_findings": [
    {"finding": "Elevated HbA1c", "significance": "Indicates poor blood sugar control over 3 months", "urgency": "soon"}
  ]
}
```

## Plain language output

```
Your blood sugar control (HbA1c) is higher than the target range. 
This means your average blood sugar over the last 3 months has been 
elevated. This is something to discuss with your doctor at your next 
visit — they may recommend adjustments to diet, exercise, or medication.

What needs attention:
- HbA1c 8.2%: What it means: Blood sugar has been running high. Action: Schedule follow-up with your doctor within 2 weeks.
```

## Audience options

- `patient` — simple language, reassuring tone, practical actions
- `family member` — same as patient but third person
- `nurse` — clinical language, prioritized action list
- `doctor` — full data, differential suggestions

## Important disclaimer

This tool is for informational purposes only — always verify findings and treatment decisions with a qualified medical professional.

## Setup

1. Jina AI API key
2. OpenAI API key
3. Slack Bot Token + #medical-alerts channel
