# conference-talk-submission-coach

Getting a talk accepted at a good conference is harder than it looks. The abstract is a sales document — it has to convince a review committee that your topic deserves 40 minutes of their attendees' time and that you can actually deliver it. Most first drafts don't do that. They describe the talk instead of selling it.

I built this after getting rejected twice from the same conference with what I thought were solid abstracts. Turns out they were descriptive, not compelling. The titles were forgettable. The committee perspective section I added to this makes the difference obvious.

You POST your draft abstract, the conference name, and some context. It searches the web for what that conference tends to select, sends everything to Claude which reviews the submission like an experienced program committee member and a veteran speaker rolled together, and returns detailed feedback: a score, a verdict, specific issues with fixes, three alternative title options, a full rewritten abstract ready to copy-paste, structure suggestions, and a paragraph written as if a committee member is reading it right now.

---

## What it does

1. Accepts a POST: talk title, abstract, conference name, format, audience level, speaker bio, outline notes, specific concerns
2. Searches Tavily for the conference's typical topics, audience, and past talk themes
3. Sends everything to Claude for review
4. Returns:
   - Overall score (1–10) and verdict: `strong_submit` / `submit_with_tweaks` / `significant_revision_needed` / `rethink_angle`
   - One-line honest summary
   - Specific strengths
   - Critical issues with exact fixes
   - Title assessment + 3 alternative titles
   - Full rewritten abstract (copy-paste ready)
   - Structure suggestions if an outline was provided
   - What would differentiate this from similar submissions
   - Committee perspective (what they'd be thinking)
   - Speaker bio feedback
5. If a speaker email is provided, delivers the formatted HTML feedback report
6. Returns everything as JSON in the webhook response

---

## Stack

- **n8n** — webhook + workflow
- **Tavily API** — conference research
- **Anthropic Claude** (claude-opus-4-5) — CFP review and rewriting
- **SMTP** — feedback delivery (optional)

---

## Setup

### 1. Get Tavily API key

Sign up at [tavily.com](https://tavily.com). Free tier is fine for typical usage.

### 2. Environment variables

```
TAVILY_API_KEY=tvly-your-key
FROM_EMAIL=cfpcoach@yourdomain.com
```

### 3. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** (optional, only if emailing feedback)

### 4. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/cfp-coach \
  -H "Content-Type: application/json" \
  -d '{
    "talk_title": "The Hidden Cost of Fast Tests",
    "abstract": "Everyone wants fast tests. We mock dependencies, stub external calls, and avoid anything slow. But in chasing speed, we often end up with test suites that pass confidently while our systems quietly rot. This talk examines the real tradeoffs between test speed and test fidelity, shows concrete examples of mocks that lie, and proposes a layered testing strategy that gives you both speed and confidence without sacrificing one for the other.",
    "outline_or_notes": "1. The seduction of the green bar (5 min)\n2. Three real bugs that tests missed — and why (10 min)\n3. The mock spectrum: what to stub, what not to (10 min)\n4. A layered strategy that actually works (10 min)\n5. Q&A",
    "conference_name": "RailsConf",
    "conference_url": "https://railsconf.org",
    "format": "full_talk",
    "audience_level": "intermediate",
    "speaker_bio": "Senior engineer at a mid-size fintech. 8 years with Rails. Have given two internal talks but this would be my first conference proposal.",
    "speaker_email": "speaker@email.com",
    "submission_deadline": "2025-05-15",
    "specific_concerns": "Not sure if the title is punchy enough. Also worried the abstract is too abstract — doesnt show enough concrete examples."
  }'
```

**Required:** `talk_title`, `abstract`, `conference_name`

**Strongly recommended:** `outline_or_notes`, `speaker_bio`, `speaker_email`

---

## Verdicts explained

| Verdict | Meaning |
|---|---|
| `strong_submit` | This is ready. Maybe a word or two to change but don't second-guess it. |
| `submit_with_tweaks` | Solid foundation, specific issues to fix before submitting. |
| `significant_revision_needed` | The core idea is there but the abstract itself needs a rewrite. |
| `rethink_angle` | The topic or framing isn't working. Consider a different approach. |

---

## The rewritten abstract

This is the most immediately useful output. Claude rewrites your abstract based on its own feedback — sharper opening, clearer takeaways, better sales language, concrete deliverables for the audience. It's roughly the same length as your original. Copy it, compare it to yours, take what works.

You don't have to use the rewrite verbatim. Most people use it as a starting point and keep their own voice in the final version.

---

## The `specific_concerns` field

If you already know what you're worried about, put it here. Claude will address those concerns directly rather than you having to infer whether they were addressed.

Examples:
- `"The title feels generic but I can't figure out how to make it specific"`
- `"I've given this talk internally — not sure how to frame it as new for external audiences"`
- `"Abstract is 300 words, conference limit is 150 — need to know what to cut"`
- `"Worried this topic has been done to death"`

---

## Without Tavily

If you don't want to use the Tavily search:
1. Delete the **Research Conference** and **Merge Context** nodes
2. In the **Claude CFP Coach** node, replace `{{ $json.confContext }}` with a static description of the conference or leave it blank
3. Connect **Valid?** → directly to **Claude CFP Coach**

You lose the conference-specific context but everything else works fine.

---

## Building a self-service tool

This works well as a self-service tool for a speaker community, meetup group, or company speaker program. Add a Tally form at the front, paste the webhook URL, and anyone in the group can get feedback on their submissions without bothering a human reviewer.

---

## Limitations

- Conference research depends on what's indexed about that conference. Big conferences (PyCon, RailsConf, KubeCon) have lots of context available. Smaller or newer conferences may have little to find — the review will still work but won't be conference-specific.
- The rewritten abstract is Claude's interpretation. It may shift the emphasis or tone in ways you don't want. Treat it as a strong reference point, not the final word.
- Claude doesn't know what other talks have been submitted to this specific conference this cycle — it can only assess what it knows from public information.

---

## Ideas

- [ ] Submission history tracking — save all reviews to a sheet, see how scores improve across iterations
- [ ] A/B mode — submit two versions of an abstract, Claude compares them and picks the stronger one
- [ ] Speaker preparation briefing — separate workflow that, once accepted, generates prep notes, potential audience questions, and timing guidance

---

## License

MIT.
