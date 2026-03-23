# FLOOWBOX - Meeting Notes to Action Items (Whisper + GPT-4o)

I used to spend 20-30 minutes after every client call writing up notes and action items. Now I upload the recording and get a structured summary in under 2 minutes.

## What it does

Upload any meeting audio file via webhook. OpenAI Whisper transcribes the recording. GPT-4o then analyzes the transcript and extracts a clean summary, action items with owners and deadlines, decisions made, and follow-ups required. Everything gets saved to Notion as a new page and emailed to all attendees automatically.

## Tools Used
- **Orchestration:** n8n
- **Transcription:** OpenAI Whisper (whisper-1)
- **Analysis:** OpenAI GPT-4o
- **Storage:** Notion database
- **Email:** SMTP
- **Trigger:** Webhook (upload audio file)

## Flow

```
Meeting Audio Upload (Webhook)
  → Whisper transcribes audio → full text
  → GPT-4o extracts:
      - Summary (3-4 sentences)
      - Action items (owner + task + deadline)
      - Decisions made
      - Follow-ups required
  → Parse structured JSON
  → Save to Notion page
  → Email summary to all attendees
```

## Output format

```json
{
  "summary": "Team discussed Q2 automation roadmap...",
  "action_items": [
    {"owner": "Navtej", "task": "Send proposal draft", "deadline": "Mar 25"},
    {"owner": "Client", "task": "Share API access", "deadline": "Mar 22"}
  ],
  "decisions": ["Approved 3-month engagement", "Starting with WhatsApp automation"],
  "follow_ups": ["Schedule technical discovery call"]
}
```

## Why I built this

Client meetings generate a lot of verbal commitments that get forgotten. Having auto-extracted action items with owners and deadlines sent to everyone immediately after the call has eliminated the "I thought you were doing that" problem completely.

## Setup

1. OpenAI API key (for both Whisper and GPT-4o)
2. Notion integration + Database ID
3. SMTP credentials for email
4. POST audio file URL to webhook endpoint

## Supported audio formats
mp3, mp4, mpeg, mpga, m4a, wav, webm — anything Whisper supports.
