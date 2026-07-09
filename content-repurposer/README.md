# content-repurposer

Most content gets written once and used once. A 2,000-word blog post has a LinkedIn post, a Twitter thread, an email newsletter, six quote cards, and a short-form video script inside it — but extracting those takes time most teams don't have. This takes any long-form source content and generates fully written versions for up to 9 target formats in a single call.

---

## What it does

Takes source content (blog post, podcast transcript, research report, case study, etc.) and a list of target formats. Claude identifies the 3–5 most repurposable insights, then writes each requested format natively — not summaries, but content that fits how each format actually works:

- **LinkedIn post** — hook line, full post body, hashtags
- **Twitter thread** — numbered tweets starting with the most provocative claim
- **Email newsletter** — subject line, preview text, full body
- **Short-form video script** — hook line (first 3 seconds), full script under 60s, caption
- **Infographic outline** — title, sections with data points
- **Slide deck outline** — slide-by-slide with key point and visual suggestion per slide
- **Quote cards** — the most shareable quotes with attribution
- **FAQ** — question and answer pairs derived from the content
- **Summary paragraph** — 150–200 word plain-language summary

Builds a formatted HTML doc showing each format in its native style.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/repurpose-content \
  -H "Content-Type: application/json" \
  -d '{
    "source_format": "blog_post",
    "brand_voice": "Direct and practical — we treat readers as smart operators, no buzzwords",
    "target_audience": "Operations managers at small companies",
    "product_name": "Flowdesk",
    "call_to_action": "Start free trial at flowdesk.com",
    "reply_email": "content@flowdesk.com",
    "target_formats": ["linkedin_post", "twitter_thread", "email_newsletter", "quote_cards"],
    "source_content": "[paste your full blog post, transcript, report, or other long-form content here — min 100 chars, up to 8000]"
  }'
```

**Required:** `source_content`, `source_format`

---

## Source formats

`blog_post`, `podcast_transcript`, `webinar_transcript`, `research_report`, `case_study`, `whitepaper`, `video_script`, `interview`

---

## Target formats

`linkedin_post`, `twitter_thread`, `email_newsletter`, `short_form_video_script`, `infographic_outline`, `slide_deck_outline`, `quote_cards`, `faq`, `summary_paragraph`

Request any subset. All are generated in a single Claude call.

---

## Native format adaptation

Each format gets written for how people actually consume content in that medium — not the same summary reformatted. A LinkedIn post opens with a hook that earns the "see more" click. A Twitter thread opens with the most interesting claim. A video script front-loads the value in the first 3 seconds before dropping into the body. The source insights are constant; the framing is native.

---

## Limitations

- Source content is capped at 8,000 characters. For longer content (full webinars, long reports), either summarize the second half in the source content field or run the agent on sections separately.
- Format quality is best when source content is substantive and opinionated. Generic content ("here are some tips about productivity") produces generic repurposed content.

---

## License

MIT.
