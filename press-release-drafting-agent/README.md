# press-release-drafting-agent

Writing a press release is one of those things that looks simple and isn't. The format is rigid. The headline has to earn placement. The lede has to deliver the news in the first 35 words. The quote has to sound like a real person said it, not like it was written by a committee. Most first drafts get all of this wrong.

This takes a brief — announcement type, key facts, any quotes you have, your company description — and produces a properly structured press release in AP Style. Headline, subheadline, dateline lede, body, quote block, boilerplate, contact info, end mark. Two alternative headlines and a short set of notes for the PR team come alongside.

Works for product launches, funding rounds, executive hires, partnerships, awards, expansions — any standard announcement type.

---

## What it does

1. Accepts a POST: company name, description, announcement type, headline idea, key facts array, quotes, target media, tone, embargo date, contact details, boilerplate
2. Claude drafts a full AP Style press release:
   - Polished headline (active voice, present tense, newsworthy)
   - Subheadline
   - FOR IMMEDIATE RELEASE or embargo line
   - Dateline
   - Lede paragraph with who/what/when/where/why in ~35 words
   - 2–3 body paragraphs with context and details
   - Executive quote block (uses provided quotes or creates placeholder)
   - Company boilerplate
   - Media contact block
   - ### end mark
3. Returns 2 alternative headline options
4. Provides PR team editor notes: what's strong, what to verify, suggested media targets
5. Builds a clean HTML output formatted like an actual press release
6. Emails if `reply_email` provided
7. Returns full JSON including the complete release text

---

## Stack

- **n8n** — webhook + workflow
- **Anthropic Claude** (claude-sonnet-4-20250514) — PR writing
- **SMTP** — optional email delivery

---

## Setup

### 1. Environment variables

```
FROM_EMAIL=pr@yourcompany.com
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** (optional)

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/draft-press-release \
  -H "Content-Type: application/json" \
  -d '{
    "company_name": "Meridian Health Tech",
    "company_description": "Meridian Health Tech builds AI-powered clinical documentation tools for outpatient practices. Its flagship product, ChartAssist, reduces physician documentation time by 40%.",
    "announcement_type": "funding",
    "headline_idea": "Meridian raises Series A to expand AI documentation platform",
    "key_facts": [
      "Raised $18M Series A led by Andreessen Horowitz with participation from General Catalyst",
      "Round brings total funding to $23M",
      "Will use funds to hire 40 engineers and expand into hospital systems",
      "ChartAssist currently used by 1,200 outpatient clinics across 34 states",
      "Reduces average documentation time from 22 minutes to 13 minutes per patient",
      "Company grew ARR 3x year-over-year"
    ],
    "quotes": [
      {
        "quote": "Physician burnout from administrative work is a crisis, not a nuisance. This funding lets us move faster to give clinicians back the time they went into medicine for.",
        "attribution": "Dr. Sandra Osei, CEO and Co-founder, Meridian Health Tech"
      },
      {
        "quote": "The documentation burden in healthcare is one of the most solvable problems in medicine. Meridian has figured out how to solve it at scale.",
        "attribution": "Vijay Pande, General Partner, Andreessen Horowitz"
      }
    ],
    "target_media": "healthcare trade press, tech business press, VC and startup media",
    "tone": "professional but accessible",
    "embargo": "FOR IMMEDIATE RELEASE",
    "contact_name": "Jamie Park",
    "contact_email": "press@meridianhealth.tech",
    "contact_phone": "+1 415 555 0147",
    "reply_email": "comms@meridianhealth.tech"
  }'
```

**Required:** `company_name`, `announcement_type`, `headline_idea`, `key_facts` (array, minimum 2 items)

---

## Announcement types

`product_launch`, `funding`, `partnership`, `acquisition`, `executive_hire`, `award`, `expansion`, `event`, `research`, `other`

The type shapes the structure — a funding announcement leads differently from a product launch.

---

## The `key_facts` array

This is the most important field. List the facts you want in the release, in any order — Claude will determine which ones lead and which support. Include:
- The core news (who, what, how much/how many, when)
- Context that makes it meaningful (growth metrics, market size, customer numbers)
- Any specific details editors will want (investors by name, geographic scope, timeline)

Minimum 2 facts, maximum 10. More specific is better — "ARR grew 3x year-over-year" beats "strong growth."

---

## Quotes

Quotes can be provided or left empty. If provided, Claude uses them verbatim (with minor cleanup if they're rough). If not provided, Claude creates a placeholder quote clearly marked `[PLACEHOLDER — NEEDS APPROVAL]`.

Format:
```json
"quotes": [
  { "quote": "The actual quote text.", "attribution": "First Last, Title, Company" }
]
```

Up to 3 quotes. First quote typically goes in the main body, additional quotes are used where appropriate.

---

## Boilerplate

The standard "About [Company]" paragraph at the bottom. If you don't provide one, Claude generates one from your `company_description`. Paste your official boilerplate if you have it — this should match what's on your website exactly.

---

## Editor notes

The response includes `editor_notes` — 2–3 observations for the PR team:
- What's strongest in the release (the most newsworthy element)
- What needs verification before sending (any facts that should be double-checked)
- Suggested media targets for this specific announcement

These don't go in the release itself. They're internal guidance.

---

## Output format

The full press release is in `draft.body` as a string with `\n\n` paragraph breaks. Copy it directly into a Word doc, email, or PR distribution platform (PR Newswire, Business Wire, etc.).

The HTML version formats it visually — useful for internal review and approval before distribution.

---

## Limitations

- Claude follows AP Style conventions but isn't infallible. Review the dateline format, any numbers (AP Style has specific rules for numerals), and quote attribution formats before sending.
- Placeholder quotes are clearly marked but you must replace them with approved quotes from a real person before distributing.
- The release is based entirely on what you provide. If a key fact is missing or wrong, the release reflects that.

---

## Ideas

- [ ] Distribution integration: pipe the final release directly to PR Newswire or Business Wire API
- [ ] Media list generator: given the release topic, generate a targeted journalist contact list
- [ ] Follow-up pitch generator: companion agent that writes a short email pitch for individual journalists based on the release
- [ ] Approval workflow: multi-step review before the release is marked ready to send

---

## License

MIT.
