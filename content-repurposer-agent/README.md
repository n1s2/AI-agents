# content-repurposer-agent

Writing a long-form piece and then separately producing a LinkedIn post, a Twitter thread, an email, and a set of pull quotes is four jobs not one. Most people either skip most of it or produce watered-down versions of the same text in different boxes.

This takes a source piece — blog post, podcast transcript, newsletter, research report, anything — and generates platform-native versions for however many formats you need in one shot. Claude doesn't just trim the content; it reshapes it. A LinkedIn post has a different grammar from a Twitter thread. A TikTok script is spoken word with a 3-second hook requirement. Each format is written as if it was created natively for that platform.

---

## What it does

1. Accepts a POST: source content, source type, title, target formats list, brand voice notes, target audience
2. Claude reads the full source, identifies the strongest angle, and generates each requested format:
   - **LinkedIn post** — 150–300 words, hook first line, 1–2 hashtags max
   - **Twitter/X thread** — 6–10 numbered tweets, each under 280 characters
   - **Email newsletter** — subject line + 200–350 word body with CTA
   - **Instagram caption** — punchy opener, 100–200 words, 5–8 hashtags
   - **YouTube description** — 150–250 words with natural keywords
   - **TikTok script** — spoken word, 45–60 seconds, hook in first 3 seconds
   - **Pull quotes** — 3–5 standalone quotes for image overlays
   - **Key takeaways** — 5–7 actionable bullet points
   - **FAQ** — 4–6 Q&A pairs based on what the content answers
3. Identifies the single best-performing angle from the source
4. Builds a clean HTML output document with each format in its own section
5. Emails if `reply_email` provided
6. Returns full JSON in webhook response

---

## Stack

- **n8n** — webhook + workflow
- **Anthropic Claude** (claude-sonnet-4-20250514) — content repurposing
- **SMTP** — optional email delivery

---

## Setup

### 1. Environment variables

```
FROM_EMAIL=content@yourdomain.com
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** (optional)

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/repurpose-content \
  -H "Content-Type: application/json" \
  -d '{
    "source_type": "blog_post",
    "source_title": "Why Most Productivity Systems Fail After 3 Weeks",
    "source_author": "Alex Kim",
    "source_content": "Every January, millions of people download a new productivity app, buy a new planner, or commit to a new morning routine. By February, most have abandoned it. This is not a willpower problem...",
    "target_formats": ["linkedin_post", "twitter_thread", "pull_quotes", "key_takeaways", "email_newsletter"],
    "target_audience": "knowledge workers, founders, people interested in productivity",
    "brand_voice": "Direct and slightly contrarian. We avoid hustle culture language. No exclamation points. Smart but accessible.",
    "include_emojis": false,
    "reply_email": "writer@company.com"
  }'
```

**Required:** `source_content`, `source_type`

---

## Source types

`blog_post`, `podcast_transcript`, `video_transcript`, `newsletter`, `webinar_notes`, `research_report`, `interview`, `thread`, `other`

The source type helps Claude calibrate how to extract content — a podcast transcript needs more condensing than a polished essay.

---

## Choosing formats

Pass any subset of the available formats. If you don't pass `target_formats`, it defaults to the first four: `linkedin_post`, `twitter_thread`, `email_newsletter`, `instagram_caption`.

All available formats:
```
linkedin_post, twitter_thread, email_newsletter, instagram_caption,
youtube_description, tiktok_script, pull_quotes, key_takeaways, faq
```

---

## Brand voice notes

The `brand_voice` field is optional but significantly improves output quality. Describe your tone in 2–4 sentences:
- What words or phrases you avoid
- Formal vs casual register
- Whether you use humor, and what kind
- Any style rules (no exclamation points, always use contractions, etc.)

Without it, Claude uses the tone of the source content as a guide.

---

## The "best angle" field

Every response includes a `best_performing_angle` — Claude's read on the single most shareable idea in the source. This is often different from the main thesis. A post about productivity systems might have a buried analogy that's more compelling than the central argument. Useful for deciding which idea to lead with if you're scheduling posts across multiple days.

---

## Source content length

The webhook accepts up to 8,000 characters of source content — roughly 5,000–6,000 words. For longer content (full book chapters, long research reports):
- Submit the most important sections
- Or split into multiple calls by section and combine the pull quotes / takeaways manually

---

## Posting the outputs

The webhook returns clean text in the JSON response. From there:
- Copy LinkedIn post directly to the LinkedIn composer
- Schedule tweets via Buffer, Typefully, or Hootsuite
- Paste the email body into Mailchimp / ConvertKit
- Feed the Instagram caption to your scheduling tool

To automate posting: add nodes after **Return Output** that call the relevant platform APIs. The content is already in the right format.

---

## Limitations

- Claude doesn't know what you've already posted. If you repurpose the same source twice, it'll generate fresh versions rather than avoiding overlap. Keep a log of what you've published.
- The TikTok script is written for spoken delivery — it reads awkwardly on paper. Read it aloud before recording.
- Pull quotes work best when the source has strong, quotable sentences. If the source is mostly data-driven, the quotes will be less punchy.
- Platform algorithm rules change. The formatting guidance (character counts, hashtag counts) was calibrated to current best practices but may drift.

---

## Ideas

- [ ] Scheduled repurposing: connect to an RSS feed or blog webhook, auto-repurpose every new post on publish
- [ ] A/B variant mode: generate two versions of the LinkedIn post with different angles
- [ ] Content calendar push: auto-schedule outputs to Buffer or Hootsuite API
- [ ] Google Docs integration: paste source from a Doc URL instead of raw text

---

## License

MIT.
