# podcast-research-briefing-agent

Good podcast interviews don't happen because the host is naturally curious. They happen because someone did two hours of research beforehand and distilled it into the right questions. Most podcast hosts either skip that step or do it manually every time.

This automates the research. You POST a guest's name, the episode topic, and some basic context. It searches the web for their background, career history, and past interviews, synthesizes everything with Claude, and delivers a structured briefing: who they are, what they actually believe, angles other podcasts haven't covered, questions at different depths, things to handle carefully, and what to avoid doing that would make the interview feel like every other one they've done.

Takes about 30 seconds to run. I use it the day before recording.

---

## What it does

1. Accepts a POST: guest name, title, company, episode topic, podcast name, audience, interview date, host info, specific focus areas
2. Runs two parallel web searches via Tavily API:
   - Guest background, career, public statements
   - Past podcast/interview appearances
3. Merges results and sends to Claude which synthesizes a full briefing:
   - TL;DR (3 sentences, written like a text to a colleague)
   - Career background snapshot
   - Key areas of genuine expertise
   - Notable non-obvious facts
   - Their known public positions
   - Unexplored angles with rationale
   - Handle with care items
   - Four question sets: openers, substantive, unexpected, closing
   - What to avoid
   - Sources consulted
4. Formats into a clean sectioned HTML email
5. Emails to the host (if email provided)
6. Returns full briefing JSON in the webhook response

---

## Stack

- **n8n** — webhook + workflow
- **Tavily API** — real-time web search
- **Anthropic Claude** (claude-opus-4-5) — research synthesis and briefing writing
- **SMTP** — optional email delivery

---

## Setup

### 1. Get a Tavily API key

Sign up at [tavily.com](https://tavily.com). Free tier allows 1,000 searches/month which is plenty for typical podcast production volume.

### 2. Environment variables

```
TAVILY_API_KEY=tvly-your-key-here
PODCAST_NAME=The Deep Work Podcast
PODCAST_AUDIENCE=knowledge workers and entrepreneurs
HOST_NAME=Alex Rivera
HOST_EMAIL=alex@podcast.com
FROM_EMAIL=research@podcast.com
```

### 3. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** — only needed if using email delivery

### 4. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/podcast-briefing \
  -H "Content-Type: application/json" \
  -d '{
    "guest_name": "Lenny Rachitsky",
    "guest_title": "Founder",
    "guest_company": "Lenny'\''s Newsletter",
    "episode_topic": "How the best PMs think about prioritization and roadmaps",
    "podcast_name": "Product Minds",
    "podcast_audience": "product managers and founders",
    "interview_date": "2025-05-06",
    "host_name": "Jamie Osei",
    "host_email": "jamie@productminds.fm",
    "specific_focus_areas": "his framework for saying no, how he thinks about company stage affecting PM strategy",
    "previous_appearances": "Lex Fridman, Tim Ferriss, a16z podcast"
  }'
```

**Required:** `guest_name`, `episode_topic`

**Strongly recommended:** `podcast_audience` — Claude uses this to calibrate question depth and assumed knowledge level

---

## The `previous_appearances` field

If you know where else the guest has appeared, list those shows. Claude uses this two ways: it avoids asking questions that are clearly covered in those episodes, and the "unexplored angles" section specifically tries to go somewhere those shows didn't.

If you leave this blank, Claude uses the Tavily search results to infer past appearances from the web.

---

## The `specific_focus_areas` field

This is the most useful field after the basic guest info. If you want to talk about a specific period in their career, a particular project, a published idea — put it here. Claude will build questions around it rather than just covering their general background.

Examples:
- `"focus on the 2022 pivot and what they learned from it"`
- `"their contrarian take on remote work culture"`
- `"specifically the chapter on pricing strategy in their book"`

---

## The unexplored angles section

This is usually the most valuable part of the briefing. Claude looks at what the guest talks about publicly and tries to identify what they know a lot about but don't get asked about — adjacent expertise, older work that's less discussed, positions they hold that contradict common wisdom.

The quality of this section depends heavily on how much is indexed about the guest. For well-known guests with lots of public content it's excellent. For emerging guests or those with limited online presence it'll be more speculative.

---

## Using without Tavily

If you don't want to use Tavily, replace the two search nodes with manual input. Add a `background_notes` field to the webhook body and pass it directly to Claude's prompt instead of search results. You lose the real-time web research but keep the Claude synthesis and question generation. Useful for guests where you already have a lot of context.

Alternatively, swap Tavily for another search API — Serper, Brave Search, or Exa all work similarly.

---

## Response time

Typically 15–25 seconds end to end:
- Tavily searches run in parallel (~3–5 seconds each)
- Claude synthesis (~10–15 seconds for a full briefing)
- Email send (~1 second)

For a webhook, that's long enough that you might want to make the request async. The briefing returns in the webhook response body — you can ignore it and just wait for the email, or store the response if you're building something on top.

---

## Building a simple form on top

For a podcast production team, a basic form is more convenient than curl. A Tally or Typeform form with 6–7 fields posting to this webhook works fine. Team members fill it out when booking a guest, briefing arrives by email automatically.

---

## Limitations

- Research quality depends on how much public information exists about the guest. Obscure or private individuals will produce sparse briefings — Claude will say so rather than making things up.
- Tavily searches return snippets, not full articles. Claude synthesizes from those snippets. For very technical guests or niche topics, the briefing benefits from the host adding supplementary context via `specific_focus_areas`.
- The "handle with care" section is based on publicly available information. It won't surface things that aren't indexed.
- This generates one briefing per call with no memory of past episodes. If you want to track past guests and avoid repeating questions across episodes, you'd need to add a Google Sheets lookup.

---

## Ideas

- [ ] Guest CRM: log all past briefings, track which questions were asked, avoid repeats across episodes
- [ ] Notion integration: push briefing directly into a Notion episode planning page
- [ ] Audio summary: text-to-speech version of the TL;DR for hosts who prefer listening while commuting
- [ ] Automatic scheduling: trigger briefing generation N days before interview date from a calendar event

---

## License

MIT.
