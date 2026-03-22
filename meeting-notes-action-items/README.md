# FLOOWBOX - Meeting Notes to Action Items with AI

Raw meeting notes in → structured action items, decisions, and summary email out. No more "who was supposed to do what?" after every call.

## What it does

Paste any meeting notes or transcript into the workflow. GPT-4o extracts a clean summary, all action items with owners and deadlines, key decisions made, and next steps. The result gets emailed automatically to all attendees.

## Tools Used
- **Orchestration:** n8n
- **AI:** OpenAI GPT-4o
- **Email:** Gmail / SMTP
- **Trigger:** Manual

## Flow
```
Manual Trigger
  → Set meeting title, date, attendees, raw notes
  → GPT-4o extracts structured summary
  → Format email
  → Send to all attendees
```

## What the AI extracts

- Meeting summary (3-4 sentences)
- Action items (task + owner + deadline)
- Key decisions made
- Next steps
- Follow-up date

## Why I built this

Every FLOOWBOX client call was ending with me manually writing up notes and sending a follow-up email. With multiple clients this was 30-45 minutes of admin after every call. Now I paste the notes and it's done in 10 seconds.

## How to use

1. After any meeting, paste your raw notes into the `raw_notes` field
2. Add attendee emails to `attendees_emails` (comma separated)
3. Run the workflow
4. Everyone gets a clean summary within seconds

## Extending this

Connect to Fireflies.ai or Otter.ai webhook to trigger automatically when a meeting recording is processed — fully zero-touch meeting documentation.
