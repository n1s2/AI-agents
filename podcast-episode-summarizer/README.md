# podcast-episode-summarizer

Publishing a podcast episode involves more than uploading audio. Show notes that actually rank and convert, a newsletter feature, a pull quote, LinkedIn and Twitter posts — most creators skip most of this because it takes too long. This generates the full content pack from a transcript or timestamped notes in a single call.

---

## What it does

Takes a transcript or producer notes, episode title, guest details, and podcast context. Claude produces:

**Show notes:** intro paragraph that sells the listen, "what you'll learn" bullets, episode highlights, resources mentioned, guest bio

**Newsletter feature:** subject line + 150–200 word feature that gives enough value to be worth reading even without listening, but makes the reader want to hear more

**Social pack:** LinkedIn post (200–250 words, insight-led), Twitter thread opener, Instagram caption with hashtag suggestions, pull quote (the single most shareable line)

**Supporting structure:** episode summary, key insights with context, suggested timestamps/segment topics

All content is specific to what was actually discussed — not generic descriptions that could apply to any episode.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Setup

**Env vars:** `FROM_EMAIL`

**Credentials:** Anthropic API (LangChain node), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/summarize-podcast \
  -H "Content-Type: application/json" \
  -d '{
    "podcast_name": "Startup Real Talk",
    "episode_number": "87",
    "episode_title": "Why 70% of B2B startups price wrong (and what actually works)",
    "guest_name": "Sofia Reyes",
    "guest_title": "Former VP Pricing at Stripe, now founder of Pricepoint",
    "host_name": "Marcus Webb",
    "target_audience": "B2B SaaS founders and product leaders",
    "tone": "conversational but substantive",
    "output_format": "all",
    "reply_email": "marcus@startupre altalk.com",
    "transcript_or_notes": "[paste full transcript or detailed timestamped notes here]"
  }'
```

**Required:** `episode_title`, `transcript_or_notes`

---

## Output format options

`show_notes` — just show notes assets
`newsletter_feature` — just the newsletter feature
`social_pack` — just social content
`all` — everything (default)

---

## Transcript vs notes

Both work. A full transcript produces richer, more specific content (including better pull quotes and specific timestamps). Detailed producer notes (bullet points of topics covered, key moments, quotes) also work well. The minimum is 100 characters — anything shorter produces generic output.

The input is capped at 12,000 characters. For longer transcripts, either pass the full thing (it'll be truncated at the end) or summarize the second half of the episode in the notes field.

---

## Limitations

- Pull quotes from notes (not verbatim transcripts) are paraphrases, not direct quotes. Don't attribute them as verbatim without checking the source audio.
- Timestamps in `timestamps_suggested` are topic labels, not timecodes — the agent doesn't have access to the audio timeline. Add actual timecodes manually after generation.

---

## License

MIT.
