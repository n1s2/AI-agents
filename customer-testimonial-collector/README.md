# customer-testimonial-collector

Most companies have happy customers who would give a testimonial if asked. They just don't get asked, or they get a generic mass email that feels transactional, or when they do respond with something heartfelt it sits in someone's inbox for three months before anyone turns it into a usable quote.

This handles the full cycle. Two webhooks: one to request a testimonial (fires a personalized outreach email referencing the customer's specific use case and results), and one to receive the submission (processes the raw text into polished, publish-ready versions for every format — website hero, case study, social, sales deck — plus a quality score and platform tags for the content library).

Everything logs to Google Sheets and marketing gets notified immediately when a high-quality testimonial comes in.

---

## What it does

**Request testimonial (POST `/request-testimonial`):**
- Takes customer name, email, product/service, use case, known success metrics
- Claude writes a personalized outreach email that references their specific experience
- Logs the request to Google Sheets
- Sends the email to the customer
- Returns confirmation

**Submit testimonial (POST `/submit-testimonial`):**
- Takes the raw testimonial text, customer info, rating, permission to publish
- Claude processes it:
  - Quality score (0–10) with notes for the marketing team
  - Polished quote: cleans up grammar/typos while preserving voice, never adds claims
  - Short pull quote: 15–25 words for social or image overlays
  - Key outcomes mentioned (structured data)
  - Emotion tags and use case tags for content library search
  - Platform versions: website hero, case study, social media, sales deck
  - Publish recommendation: feature prominently / use as is / minor edit needed / needs clarification / do not use
- Saves everything to Google Sheets
- Notifies marketing team with the polished quote, recommendation, and platform versions
- Returns all processed versions as JSON

---

## Stack

- **n8n** — two webhooks
- **Google Sheets** — requests log + testimonials library
- **Anthropic Claude** (claude-sonnet-4-20250514) — email writing + testimonial processing
- **SMTP** — outreach email + marketing notification

---

## Setup

### 1. Create the Google Sheet

Two tabs:

**Tab: Requests** — columns:
```
customer_id | customer_name | customer_email | customer_company | product_or_service | requested_at | status
```

**Tab: Testimonials** — columns:
```
testimonial_id | submitted_at | customer_id | customer_name | customer_company | product_or_service | rating | raw_testimonial | polished_quote | short_pull_quote | quality_score | publish_recommendation | permission_to_publish | use_case_tags | emotion_tags
```

### 2. Environment variables

```
TESTIMONIAL_SHEET_ID=your_google_sheet_id
TESTIMONIAL_FORM_URL=https://your-form-url.com
FROM_EMAIL=success@yourcompany.com
MARKETING_EMAIL=marketing@yourcompany.com
```

### 3. Credentials

- **Google Sheets OAuth2**
- **Anthropic API** (LangChain node)
- **SMTP**

### 4. Import and activate

Import `workflow.json`, activate. Two webhook URLs appear — one for requests, one for submissions.

---

## Requesting a testimonial

```bash
curl -X POST https://your-n8n.com/webhook/request-testimonial \
  -H "Content-Type: application/json" \
  -d '{
    "customer_name": "Priya Sharma",
    "customer_email": "priya@techcorp.io",
    "customer_title": "Head of Engineering",
    "customer_company": "TechCorp",
    "product_or_service": "DataSync Pro",
    "use_case": "Automating data pipelines between their warehouse and 6 downstream tools",
    "success_metric": "Reduced pipeline failures by 80% in first month",
    "requested_by": "Alex from Customer Success",
    "submission_url": "https://yourcompany.com/share-your-story"
  }'
```

**Required:** `customer_name`, `customer_email`, `product_or_service`

The outreach email Claude writes references their specific use case and known results — it reads like a genuine personal request, not a mass ask. The `requested_by` field adds the CSM's name to make it feel even more personal.

---

## Submitting a testimonial

This endpoint is what your submission form calls when the customer fills it out:

```bash
curl -X POST https://your-n8n.com/webhook/submit-testimonial \
  -H "Content-Type: application/json" \
  -d '{
    "customer_name": "Priya Sharma",
    "customer_title": "Head of Engineering",
    "customer_company": "TechCorp",
    "product_or_service": "DataSync Pro",
    "raw_testimonial": "We were spending 3-4 hours a week firefighting broken pipelines. Since switching to DataSync Pro the failures dropped massively and my team actually trusts the data now. I wish we had done this two years ago honestly.",
    "rating": 5,
    "use_case": "Data pipeline automation",
    "permission_to_publish": true,
    "preferred_platforms": ["website", "linkedin", "g2"]
  }'
```

**Required:** `raw_testimonial`, `customer_name`

---

## What the processing produces

For the above raw testimonial, Claude would output something like:

```json
{
  "quality_score": 8,
  "quality_notes": "Strong specificity on the time savings, authentic voice, clear before/after. Missing company size context that would add credibility for enterprise buyers.",
  "polished_quote": "We were spending 3-4 hours a week firefighting broken pipelines. Since switching to DataSync Pro, failures dropped dramatically and my team actually trusts the data now. I wish we'd done this two years ago.",
  "short_pull_quote": "My team actually trusts the data now — DataSync Pro changed how we work.",
  "key_outcomes_mentioned": ["3-4 hours/week saved", "significant reduction in pipeline failures", "improved team trust in data"],
  "platform_versions": {
    "website_hero": "DataSync Pro eliminated our weekly pipeline firefighting and gave my engineering team back their confidence in the data.",
    "case_study": "As Head of Engineering at TechCorp, Priya Sharma's team was spending 3-4 hours every week responding to broken data pipelines. After switching to DataSync Pro, failures dropped dramatically. 'I wish we'd done this two years ago,' she says.",
    "social_media": "We used to spend 3-4 hrs/week on broken pipelines. @DataSyncPro fixed that. My team actually trusts our data now.",
    "sales_deck": "\"My team actually trusts the data now.\" — Priya Sharma, Head of Engineering, TechCorp"
  },
  "publish_recommendation": "feature_prominently"
}
```

---

## The `permission_to_publish` field

Set to `true` by default. The field is tracked in the sheet. Before publishing any testimonial, verify this is `true`. Do not publish without explicit permission — even if the customer said something positive in a support conversation.

---

## Quality score guide

| Score | Meaning |
|---|---|
| 9–10 | Exceptional — specific outcomes, authentic voice, credible attribution |
| 7–8 | Strong — use as is or with minor polish |
| 5–6 | Usable — generic but positive, consider for volume use cases |
| 3–4 | Weak — vague, too short, or missing substance |
| 1–2 | Unusable — too short, inappropriate, or unpublishable |

---

## Building the submission form

The `/submit-testimonial` webhook accepts a POST. Build your form with:
- Tally.so, Typeform, or a custom form
- Fields: name, title, company, testimonial (long text), rating (1–5), permission checkbox
- On submit: POST to the webhook URL

Or use the webhook directly in your customer portal — trigger it when a customer completes an in-app NPS or CSAT survey.

---

## Testimonial library

Every processed testimonial is saved to the Testimonials sheet with:
- `use_case_tags` — for filtering by product area or customer type
- `emotion_tags` — for matching tone to campaign needs
- `quality_score` — for quick filtering to high-value testimonials
- All four platform versions ready to copy

The sheet becomes your searchable testimonial library. When the design team needs a quote for a landing page redesign, they can filter by tag and quality score in seconds.

---

## Limitations

- Claude polishes only — it doesn't invent or embellish. If the raw testimonial is vague ("Great product!"), the polished version will still be vague. The quality score will reflect this.
- The `preferred_platforms` field is captured and tagged, but the workflow doesn't automatically submit to third-party review platforms (G2, Capterra, Trustpilot). That's a separate integration.
- The request email is text-only. If your brand requires HTML emails with specific styling, replace the SMTP node content with formatted HTML.

---

## Ideas

- [ ] Follow-up sequence: if no submission after 7 days, send a gentle reminder via a second webhook
- [ ] G2/Capterra integration: if preferred_platforms includes a review site, send a direct link to that platform's review page
- [ ] Video testimonial variant: adapt the request email to ask for a 60-second Loom video instead of written text
- [ ] Quarterly testimonial report: aggregate quality scores and tag distributions across all testimonials collected

---

## License

MIT.
