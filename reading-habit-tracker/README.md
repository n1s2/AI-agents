# reading-habit-tracker

I read a lot but I'm bad at remembering what I read. I tried Goodreads, I tried Notion templates, I tried a plain text file. None of them stuck because the friction of opening a separate app mid-session was enough for me to just not do it.

This is a webhook I call from a shortcut on my phone. I paste in what I read today, how many pages, and a quick note if something stood out. It saves to a Google Sheet, calculates my streak and pace, estimates when I'll finish the book, and — if I wrote a note — Claude reads it and asks me one follow-up question that usually makes me think about the chapter differently.

That last part is the bit I didn't expect to actually use but do.

---

## What it does

- Accepts a POST with: book title, author, pages read, current page, notes, and whether you finished
- Saves the session to Google Sheets
- Loads your full reading history from the same sheet
- Calculates: streak, total pages all time, books completed, avg pages/session (last 14 days), days to finish estimate
- If you included notes: sends them to Claude which reflects back the key idea and asks one follow-up question
- Returns everything as JSON — progress bar, stats, streak alert, insight

---

## Stack

- **n8n** — webhook + workflow
- **Google Sheets** — session storage and history
- **Anthropic Claude** (claude-opus-4-5) — reading companion (only fires when notes are provided)

---

## Setup

### 1. Create the Google Sheet

Make a sheet called **Sessions** with these columns (exact names, case-sensitive):

```
user_id | session_date | book_title | author | pages_read | current_page | total_pages | notes | rating | finished | logged_at
```

Copy the Sheet ID from the URL and set it as `READING_SHEET_ID`.

### 2. Environment variables

```
READING_SHEET_ID=your_google_sheet_id
```

### 3. Credentials

- **Google Sheets OAuth2** — set up via n8n's Google credential flow
- **Anthropic API** — add to the LangChain node

### 4. Import and activate

Import `workflow.json`, activate, copy the webhook URL from the trigger node.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/reading-log \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "alex",
    "book_title": "The Remains of the Day",
    "author": "Kazuo Ishiguro",
    "pages_read": 42,
    "current_page": 180,
    "total_pages": 258,
    "notes": "Stevens keeps rationalizing his complicity. He calls it dignity but it reads more like cowardice dressed up in professionalism.",
    "session_date": "2025-04-24"
  }'
```

All fields except `user_id`, `book_title`, `author`, and `pages_read` are optional.

---

## Example response

```json
{
  "status": "logged",
  "session": {
    "book": "The Remains of the Day",
    "author": "Kazuo Ishiguro",
    "pagesRead": 42,
    "date": "2025-04-24"
  },
  "stats": {
    "streak": 6,
    "totalPagesAllTime": 3847,
    "booksCompleted": 11,
    "avgPagesPerSession": 38,
    "daysToFinish": 2
  },
  "progress": "[████████████████░░░░] 79%",
  "finishEstimate": "At your current pace (~38 pages/session), you'll finish in ~2 days.",
  "streakAlert": "🔥 6-day reading streak",
  "insight": "The gap you're noticing between Stevens's framing and what his actions actually reveal is the engine of the whole novel — Ishiguro never tells you directly, he just lets the self-justifications accumulate. What do you think Stevens actually believes about himself vs. what he shows the reader?"
}
```

---

## Multi-user support

The `user_id` field is how different people's data stays separate in the same sheet. If you're building this for yourself only, just hardcode a consistent value. If you want multiple users, you'd add a simple auth layer in front of the webhook (API key header check, for example).

---

## iOS/Android shortcut

On iPhone I use a Shortcuts automation that prompts me for book title, pages, and notes, then fires the POST and shows me the response. It takes about 20 seconds to log a session. If there's interest I'll add the Shortcut export to this repo.

---

## Limitations

- The streak calculation is based on calendar days. If you read twice in one day it counts as one streak day.
- Claude only fires when `notes` is 20+ characters. Short notes like "good chapter" won't trigger it — which is intentional, there's nothing interesting to ask about.
- Google Sheets isn't ideal for large reading histories (1000+ sessions) — you'd want a real database at that point, but for personal use it's fine.
- The "days to finish" estimate uses a 14-day rolling average. If you haven't read in 2 weeks it'll use just today's session count.

---

## Things I want to add

- [ ] Monthly summary email (pages read, books finished, best streak)
- [ ] `GET /reading-stats/:user_id` endpoint to pull stats without logging
- [ ] Goodreads sync (mark book as read when `finished: true`)
- [ ] Year-in-review report generation

---

## License

MIT.
