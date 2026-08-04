# customer-story-writer

Customer case studies written from a 30-minute interview usually turn into marketing copy that sounds nothing like the actual customer — generic praise with the company logo swapped in. This takes raw interview notes and produces multiple content formats simultaneously (case study, short testimonial, video script, one-pager, blog post) while never fabricating quotes, numbers, or details beyond what's in the notes.

---

## What it does

Takes customer name, company details, product, and interview notes (up to 4,000 characters). Claude extracts and generates:

- **Extracted facts** — problem before, solution used, key results, best quotes (pulled directly from notes, not invented)
- **Case study** — title, subtitle with headline result, company snapshot, challenge/solution/results in narrative form, pull quotes, and summary stat cards
- **Testimonial (short)** — a punchy 1–3 sentence quote with attribution
- **Video script** — hook, talking points in order, and suggested sound bites to capture on camera
- **One-pager** — headline, summary, bullet results, quote
- **Blog post** — title, intro, full body, CTA

Request any subset of formats. All are generated from the same interview notes in a single call.

HTML output renders each requested format with case study stat cards, pull quote blocks, and format-appropriate styling.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/write-customer-story \
  -H "Content-Type: application/json" \
  -d '{
    "customer_name": "Tanya Okonkwo",
    "customer_title": "VP Operations",
    "customer_company": "Beacon Logistics",
    "company_size": "85 employees",
    "industry": "Logistics",
    "product_name": "Flowdesk",
    "use_case": "Replacing spreadsheets for cross-warehouse task coordination",
    "key_metrics": "Reduced weekly planning meeting from 3 hours to 20 minutes. Rolled out to 60 people across 4 warehouses in 3 weeks.",
    "story_formats": ["case_study", "testimonial_short", "one_pager"],
    "reply_email": "marketing@flowdesk.com",
    "interview_notes": "Tanya said before Flowdesk, she spent every Sunday evening preparing for the Monday planning meeting, going through 4 different spreadsheets maintained by warehouse managers in Chicago, Dallas, Memphis, and LA. She said quote this is not what I was hired to do end quote. The meeting itself took 3 hours every Monday and half the time was just people getting confused about which version of the spreadsheet was current. After implementing Flowdesk, she said the weekly meeting is now 20 minutes and is mostly just confirming priorities rather than reconstructing what happened. She specifically mentioned that Marcus, the Dallas warehouse manager, was the most skeptical at first because he'd built a complex Excel macro system, but he became one of the biggest advocates within a month because it saved him time too. Rollout took 3 weeks for all 60 people across the 4 locations, which she said was faster than she expected because the interface was intuitive enough that most people didn't need training beyond a single 30 minute session. She said her favorite feature is the automatic escalation when a task hasn't been touched in 48 hours - this catches things that used to fall through the cracks. When asked about ROI she said it's hard to put a dollar number on it but she estimates she personally got back 3-4 hours per week and thinks the org-wide time savings across managers is probably 15-20 hours per week when you add it all up."
  }'
```

**Required:** `customer_name`, `interview_notes`

---

## Story formats

`case_study`, `testimonial_short`, `video_script`, `one_pager`, `blog_post`

Request one or all. Each pulls from the same underlying interview notes but is structured natively for its format.

---

## Never fabricates

Claude is explicitly instructed to use only facts, quotes, and numbers present in the interview notes. If the notes don't include a specific ROI figure, the case study won't invent one — it will note the qualitative impact instead (as in the example, where the exact ROI was described as hard to quantify but estimated). This keeps the content honest and legally safe.

---

## Extracted facts

The `extracted_facts` field surfaces what Claude identified as the core narrative before formatting into any specific content type — problem before, solution used, key results, best quotes. Useful for reviewing what the interview actually contained before it's shaped into marketing content.

---

## Limitations

- Content quality is proportional to interview note detail. A thin interview ("customer likes the product, saves time") produces thin content. Detailed notes with specific anecdotes, named people, and numbers produce compelling, specific content.
- All customer-facing content should go through the customer for approval before publishing — quotes and specific numbers need explicit sign-off.

---

## License

MIT.
