# cold-email-personalizer

Most cold emails fail for the same reasons: they open with "I hope this finds you well," spend the first paragraph talking about the sender's company, make a vague claim about value, and close with a request for a 30-minute call. Nobody replies to those because they don't feel like they were written for the person reading them.

This generates cold emails that are specific, short, and calibrated to the prospect. It optionally searches the web for recent news about the prospect and their company to find a genuine personalization hook. Claude picks the best angle, writes a subject line that earns an open, and a body that gets to the point in under 100 words. It also rates the personalization quality honestly and tells you what to research before sending.

---

## What it does

1. Accepts a POST: prospect name, title, company, LinkedIn URL (optional), prospect context, sender details, value prop, social proof, email goal, tone, word limit, phrases to avoid
2. If LinkedIn URL or prospect context is provided: searches Tavily for recent news about the prospect and company
3. Claude writes:
   - Subject line + 2 alternatives
   - Personalization hook (what specific thing opened the email)
   - Complete email body under the word limit
   - Full email with greeting and sign-off
   - Personalization quality rating (highly personalized / moderately / generic / could not personalize)
   - What to research before sending (specific items for this prospect)
   - Follow-up angle if this email doesn't get a reply
4. Builds HTML output with the email, subject alternatives, and research notes
5. Emails if `reply_email` provided
6. Returns full JSON

---

## Stack

- **n8n** — webhook + workflow
- **Tavily API** — prospect research (optional, fires when LinkedIn URL or context is provided)
- **Anthropic Claude** (claude-sonnet-4-20250514) — email writing
- **SMTP** — optional email delivery

---

## Setup

### 1. Environment variables

```
TAVILY_API_KEY=tvly-your-key
FROM_EMAIL=outreach@yourcompany.com
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** (optional)

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/personalize-email \
  -H "Content-Type: application/json" \
  -d '{
    "prospect_name": "Tanya Okonkwo",
    "prospect_title": "VP of Operations",
    "prospect_company": "Meridian Logistics",
    "prospect_context": "Meridian just announced their expansion into Southeast Asian markets in April. Tanya posted on LinkedIn last week about the challenges of onboarding new warehouse teams in unfamiliar markets.",
    "sender_name": "Jake Reyes",
    "sender_title": "Head of Partnerships",
    "sender_company": "Flowtrack",
    "value_proposition": "Flowtrack cuts new warehouse team onboarding time by 60% by replacing paper-based SOPs with interactive digital workflows that work offline. Teams are fully productive in 3 days instead of 2 weeks.",
    "social_proof": "Used by DHL and Maersk for new site onboarding. Median time-to-productivity went from 11 days to 4 days in their pilots.",
    "email_goal": "book_meeting",
    "tone": "direct",
    "word_limit": 90,
    "cta": "Would a 15-minute call this week make sense?",
    "avoid_phrases": ["I hope this finds you well", "touching base", "synergize", "circle back"],
    "reply_email": "jake@flowtrack.com"
  }'
```

**Required:** `prospect_name`, `prospect_company`, `sender_name`, `value_proposition`, `email_goal`

---

## Email goals

| Goal | CTA calibration |
|---|---|
| `book_meeting` | Calendar link or "does X time work?" |
| `get_reply` | Low-friction question — "is this on your radar?" |
| `share_resource` | "Thought you'd find this useful" with link |
| `partnership` | Mutual benefit framing, request for conversation |
| `sales_intro` | Value-first, light ask |
| `investor_intro` | Warm, concise, clear what you're raising |
| `job_inquiry` | Why you, why them, what you can contribute |
| `other` | Claude infers appropriate CTA from context |

---

## Tone options

`direct` — confident, no filler, gets to the point sentence one
`warm` — friendly and human, reads like a colleague
`formal` — conservative, good for finance or legal recipients
`casual` — short, punchy, like a text from someone who knows them
`challenger` — opens with a provocative observation to earn attention

---

## Personalization hierarchy

Claude looks for hooks in this order:
1. **Specific recent event** — company news, funding, expansion, product launch in the last 90 days
2. **Personal content** — something the prospect wrote, said, or was quoted on recently
3. **Shared challenge** — if the industry or role has a known current challenge, lead with it
4. **Company milestone** — growth, award, geographic expansion
5. **No hook available** — Claude writes a strong industry-context opener and flags it as generic

The `personalization_quality` field tells you which level was achieved so you know how much to trust the opener.

---

## The prospect context field

The most direct way to improve personalization without web search. Paste in:
- A LinkedIn post they wrote recently
- A quote from a press release or interview
- Something you noticed about their company
- Any research you've already done

Claude uses this alongside (or instead of) the web search.

---

## What to research before sending

Every output includes a "research before sending" list — specific things to look up for this particular prospect before hitting send. Not "check their LinkedIn" but "verify the Southeast Asia expansion timeline is still active, since the announcement was 3 months ago" or "confirm Tanya is still in this role — the company went through a restructure recently."

---

## The follow-up angle

If the first email doesn't get a reply, the output includes a different angle for the follow-up. Not a "just checking in" — a genuinely different hook or framing that opens a different door.

---

## Bulk personalization

For personalized outreach across a prospect list:
1. Store prospects in Google Sheets
2. Use an n8n Google Sheets trigger or loop to read each row
3. Call this webhook per prospect
4. Store the outputs back in the sheet

The webhook handles one prospect per call. For a list of 50, that's 50 calls — manageable with a 1-second delay between calls to stay within API rate limits.

---

## Without Tavily

If neither `prospect_linkedin_url` nor `prospect_context` is provided, the workflow skips the research step and goes directly to writing. Claude uses your value proposition and whatever industry context it has to write the best email it can and rates it as `generic_industry` or `could_not_personalize`.

This is still more useful than a template — the structure and CTA calibration still apply.

---

## Phrases to avoid

Pass phrases your team has decided to avoid brand-wide in `avoid_phrases`. Claude will not use them in the email. Common ones to exclude:
- "I hope this finds you well"
- "touching base"
- "circling back"
- "I wanted to reach out"
- "synergy"
- "leverage" (when used as a verb)

---

## Limitations

- Tavily searches indexed web content from the last 90 days. If a prospect has no public presence or the relevant news is older, the personalization hook will be thin. Use `prospect_context` to supplement.
- The email is written for one specific prospect. For a list, run the webhook once per person — don't try to use a single email for multiple prospects with find-and-replace.
- Word limits are guidelines. Claude may go slightly over or under the limit for readability. If strict limits are needed (LinkedIn InMail has a 300-char limit for connection requests), add explicit enforcement in the prompt.

---

## Ideas

- [ ] A/B variant: generate two emails with different hooks, test which gets better reply rates
- [ ] Reply classifier: a companion webhook that reads inbound replies and categorizes them (interested / not now / unsubscribe / referral)
- [ ] Sequence builder: generate a 3-touch sequence (email 1, follow-up 1, follow-up 2) as a single call
- [ ] CRM integration: pull prospect data from HubSpot/Salesforce, push the generated email back as a draft or note

---

## License

MIT.
