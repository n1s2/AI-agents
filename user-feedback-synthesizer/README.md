# user-feedback-synthesizer

Reading 50 individual feedback responses takes time and produces inconsistent takeaways depending on who reads them. The loudest voices get over-indexed. Common patterns get missed. Feature requests get logged without understanding what need they actually represent.

This synthesizes a batch of user feedback items into structured product intelligence: themes with frequency and severity ratings, pain points ranked by impact, what users love, top feature requests with the underlying need they represent, recommended priorities, and a watch list of signals that are low frequency now but worth monitoring.

---

## What it does

1. Accepts 3–100 feedback items (text, optionally with source, rating, date, user tier) plus product area, period, context, and known issues
2. Calculates average rating if ratings are provided
3. Claude synthesizes:
   - Overall sentiment (positive / mostly positive / mixed / mostly negative / negative)
   - 3–4 sentence summary of what the feedback picture actually means
   - Themes with frequency (high/medium/low), severity (critical/significant/moderate/minor), user language quote, underlying need, and product implication
   - Pain points ranked by impact with affected user types
   - Delight moments in users' own language
   - Top feature requests — each one analyzed for whether it's a genuine standalone request or a symptom of a deeper unmet need
   - New signals vs known issues (what's confirming things you already knew vs what's new)
   - Recommended priorities (P0/P1/P2) with rationale grounded in the feedback
   - Watch list
4. Builds HTML report with theme cards, ranked pain list, priority action plan
5. Emails if `reply_email` provided
6. Returns full JSON

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP.

---

## Setup

**Env vars:** `FROM_EMAIL`
**Credentials:** Anthropic API (LangChain node), SMTP.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/synthesize-feedback \
  -H "Content-Type: application/json" \
  -d '{
    "product_area": "Onboarding flow",
    "feedback_period": "May 2025",
    "audience": "product_team",
    "product_context": "We redesigned the onboarding flow in March. The new flow has 5 steps instead of 3 but each step is more focused. We added an optional team invite step that about 30% of users skip.",
    "known_issues": ["Email verification step takes too long on some domains", "Mobile layout on step 3 has a scroll bug (ticket ENG-4421)"],
    "reply_email": "pm@company.com",
    "feedback_items": [
      {"text": "The setup felt really long. I just wanted to try the product and had to go through 5 different screens.", "source": "app_store", "rating": 3, "user_tier": "free"},
      {"text": "I didn't understand why you were asking me to invite my team before I even knew if the product was right for me.", "source": "intercom", "user_tier": "free"},
      {"text": "The progress bar at the top helped me know how much was left. Nice touch.", "source": "nps_survey", "rating": 5},
      {"text": "Verification email took 15 minutes to arrive. Almost gave up.", "source": "support_ticket", "rating": 2},
      {"text": "I skipped the team invite step because I wasn't sure if I was going to keep using it. Came back later and couldn't find where to add people.", "source": "intercom", "rating": 3},
      {"text": "Really clear, each step made sense. Took about 4 minutes total.", "source": "nps_survey", "rating": 5},
      {"text": "Why do I need to set up a workspace name before I can see what the product does? Put the demo first.", "source": "app_store", "rating": 2},
      {"text": "The connection to Slack worked first try which I was not expecting. Usually these integrations are painful.", "source": "intercom", "rating": 5}
    ]
  }'
```

**Required:** `feedback_items` (array, min 3), `product_area`

---

## Feedback item formats

Simple strings:
```json
"feedback_items": ["feedback text 1", "feedback text 2"]
```

Structured objects:
```json
"feedback_items": [
  {"text": "feedback text", "source": "app_store", "rating": 4, "date": "2025-05-10", "user_tier": "pro"}
]
```

Mix both formats in the same array — the validator handles either.

---

## Underlying need analysis

Feature requests are often proxies for unmet needs. "Add a dark mode" might be a symptom of "I use this at night and the screen hurts my eyes" — a different solution than just implementing dark mode. Claude analyzes each feature request for what it actually represents, so the product team can decide whether to build the requested feature or address the underlying need differently.

---

## Known issues filter

Pass your current known issues list in `known_issues`. Claude uses this to distinguish:
- **Confirming known**: feedback that adds evidence or context to issues you're already tracking
- **New signals**: patterns that aren't in your known issues list

This prevents known bugs from dominating the synthesis and keeps focus on what's genuinely new.

---

## Sentiment breakdown

The `sentiment_breakdown` field counts positive, negative, and mixed responses. "Mixed" means a single response that contains both positive and negative elements (e.g., "Love the product but the setup took way too long"). The overall sentiment rolls these up.

---

## Minimum feedback items

Minimum 3 items — fewer than that produces unreliable synthesis. Meaningful results start at around 10–15 items. For large batches (100+), consider splitting by segment (free vs paid, new vs returning) and running separate syntheses for each.

---

## Limitations

- Synthesis quality depends on feedback quality. One-word responses ("bad", "okay") produce thin themes. Detailed written feedback produces richer insights.
- Frequency counts per theme reflect what's in the batch you submit, not the actual frequency across your full user base. A batch of 20 app store reviews may be biased toward users who had strong negative experiences.
- For formal research synthesis (user interviews, structured usability tests), this is a starting point, not a replacement for proper qualitative research methodology.

---

## Ideas

- [ ] App store integration: pull reviews directly from App Store Connect and Google Play Console
- [ ] Zendesk/Intercom trigger: run synthesis automatically when a new tag is applied to a batch of tickets
- [ ] Trend tracking: compare themes across monthly syntheses to track whether issues are resolved or growing
- [ ] Segment comparison: synthesize separately for free vs paid, new vs returning, to find segment-specific patterns

---

## License

MIT.
