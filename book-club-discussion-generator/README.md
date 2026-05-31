# book-club-discussion-generator

Most book club discussion guides are a list of 20 questions that nobody gets through. They start with plot recap ("what happened in chapter 3"), stay at the surface, and the conversation either stalls or gets hijacked by one person who did an English degree.

This generates a proper structured guide: questions calibrated to the group's level, organized by arc (opening → core → thematic → closing), with a timed meeting structure, facilitator prompts for each segment, specific passages worth reading aloud, tips for this book specifically, and a "what to do if energy drops" fallback. It also searches the web for critical context on the book so the questions can go deeper than general literary analysis.

Works for book clubs at any level — high school through graduate literary. Give it your group size and meeting length and it paces the session accordingly.

---

## What it does

1. Accepts a POST: book title, author, genre, group size, meeting duration, discussion level, specific themes to explore, avoid-spoilers flag, group description
2. Searches Tavily for literary criticism, themes, and analysis of the book
3. Claude generates a complete discussion guide:
   - Overview of what makes this book rich for discussion
   - Content warnings for facilitators
   - Timed meeting structure with exact facilitator prompts per segment
   - Four question sets: Opening / Core / Thematic / Closing — each with purpose and follow-up
   - Passages worth reading aloud with notes on why
   - Facilitator tips specific to this book
   - "If conversation stalls" fallback prompt
   - Recommended pairings (books, films, essays)
4. Builds a clean sectioned HTML guide
5. Emails if `reply_email` provided
6. Returns full JSON in webhook response

---

## Stack

- **n8n** — webhook + workflow
- **Tavily API** — book research
- **Anthropic Claude** (claude-sonnet-4-20250514) — discussion guide writing
- **SMTP** — optional email delivery

---

## Setup

### 1. Environment variables

```
TAVILY_API_KEY=tvly-your-key
FROM_EMAIL=bookclub@yourdomain.com
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** (optional)

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/book-club \
  -H "Content-Type: application/json" \
  -d '{
    "book_title": "Pachinko",
    "author": "Min Jin Lee",
    "genre": "historical fiction",
    "publication_year": "2017",
    "group_size": 9,
    "meeting_duration_minutes": 90,
    "discussion_level": "general_adult",
    "specific_themes": "identity, generational trauma, the immigrant experience, shame and ambition",
    "avoid_spoilers": false,
    "group_description": "mixed-age library book club, most members read for pleasure not study, two members have Korean heritage",
    "reply_email": "facilitator@library.org"
  }'
```

**Required:** `book_title`, `author`

---

## Discussion levels

| Level | Calibration |
|---|---|
| `high_school` | Accessible vocabulary, focus on character and plot before theme, relatable questions |
| `undergraduate` | Literary terminology, close reading, some critical theory |
| `general_adult` | Thoughtful but accessible, connects to lived experience, no assumed academic background |
| `literary` | Deep textual analysis, authorial intent, intertextuality, genre conventions |
| `professional` | For book clubs in professional contexts — connects themes to work, leadership, ethics |

The same book at `general_adult` vs `literary` produces quite different questions.

---

## The four question arcs

**Opening** — Low stakes, everyone can answer. Typically about first impressions, a character moment, or a sensory detail. Gets people talking before asking them to interpret.

**Core** — The main event. Usually 4–6 questions probing the book's central tensions, character choices, and narrative craft. Each includes a discussion note for when the group gets stuck.

**Thematic** — Lifts the conversation beyond the text. Connects the book's ideas to history, current events, or the readers' own experience. The best questions here have no right answer.

**Closing** — Personal and reflective. "What will you carry from this book?" or "Which character do you think made the most defensible choice?" Ends the session with energy rather than trailing off.

---

## Meeting structure

The guide generates a timed structure that fits your `meeting_duration_minutes`. For a 90-minute session it might be: 10 min opening/setup, 20 min opening questions, 35 min core questions, 15 min thematic questions, 10 min closing. The facilitator gets an exact prompt for how to open each segment.

---

## Content warnings

For books with heavy themes (trauma, violence, racism, grief, sexual content), the guide includes content warnings in a highlighted section at the top. Useful for facilitators who want to prepare the group or let members opt out of certain discussions.

---

## The `group_description` field

Optional but significantly improves the output. Note anything relevant: age range, whether members have personal connections to the book's subject matter, how experienced the group is, whether they tend toward long digressions. Claude uses this to calibrate the questions and facilitator tips.

---

## Without Tavily

Remove the **Research Book** and **Merge Context** nodes, connect **Valid?** directly to **Claude Discussion Writer**, and replace `{{ $json.bookContext }}` with an empty string. Works well for widely-known books where Claude has strong training knowledge. May be thinner for debut novels or obscure titles.

---

## Using the guide

The HTML guide is designed to be emailed to the facilitator 24–48 hours before the meeting. They can read through it, pick the questions that feel right for their group (nobody uses all of them), and adjust timing based on how talkative the group tends to be.

The JSON response includes the full structured data — useful if you're building this into a book club platform or app and want to render it differently.

---

## Limitations

- Claude's knowledge of books varies by how widely discussed they are. Bestsellers, prize winners, and canonical texts get richer questions. Debut novels from small presses may get more generic output — the Tavily research helps but depends on what's indexed.
- The meeting structure assumes a single 90-minute (or configured) session. For multi-session discussions of very long books, add a `session_number` field and prompt Claude to focus on specific chapters.
- Content warnings are based on Claude's knowledge of the book. They may miss specific passages or be incomplete for books with complex or layered trauma. Facilitators should always preview for their specific group.

---

## Ideas

- [ ] Series mode: generate guides for all books in a series with continuity references between sessions
- [ ] Author notes integration: if the book has an author Q&A or reading group guide at the back, incorporate it
- [ ] Voting integration: after each meeting, members vote on the next book — trigger this webhook automatically from the vote result
- [ ] Meeting summary: a companion webhook where the facilitator submits notes after the meeting and Claude writes a summary to share with the group

---

## License

MIT.
