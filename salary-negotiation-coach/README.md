# salary-negotiation-coach

Most people leave money on the table in salary negotiations — not because they're bad at negotiating, but because they don't know what number to ask for, don't know what to say, and capitulate at the first sign of pushback.

This takes your situation — the offer amount, the role, your experience, any competing offers, your target — searches the web for current market rate data, and gives you a complete negotiation playbook: a specific counter-offer number with rationale, the exact words to say (call script and email draft), how to respond to the three most likely pushbacks, what to negotiate if base salary is truly stuck, and the one thing not to do in your specific scenario.

It's direct. It tells you what to ask for, not "consider negotiating."

---

## What it does

1. Accepts a POST: job title, offer amount, currency, scenario, target, market data, experience, current salary, competing offer, other comp details, negotiation style, specific concerns
2. Searches Tavily for current market salary data for the role/location/industry
3. Claude provides a complete coaching package:
   - Offer assessment: strong / fair / below market / significantly below market
   - Market range estimate with gap analysis
   - Specific counter-offer amount with best/realistic/walk-away numbers
   - Exact words to say — a call script and full email draft
   - 3 likely pushbacks with specific word-for-word responses
   - Beyond-base-salary negotiation items if base is stuck (equity, signing bonus, PTO, remote flexibility, etc.)
   - Competing offer strategy (if applicable)
   - Timing advice
   - Red flags in the situation
   - The one thing not to do in this specific scenario
4. Builds a clean HTML coaching report with all sections
5. Emails if `reply_email` provided
6. Returns full JSON

---

## Stack

- **n8n** — webhook + workflow
- **Tavily API** — market rate research
- **Anthropic Claude** (claude-sonnet-4-20250514) — negotiation coaching
- **SMTP** — optional email delivery

---

## Setup

### 1. Environment variables

```
TAVILY_API_KEY=tvly-your-key
FROM_EMAIL=coach@yourdomain.com
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** (optional)

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/negotiate-salary \
  -H "Content-Type: application/json" \
  -d '{
    "job_title": "Senior Product Manager",
    "company": "Stripe",
    "industry": "fintech",
    "location": "San Francisco, CA",
    "scenario": "initial_offer",
    "offer_amount": 185000,
    "offer_type": "annual_salary",
    "currency": "USD",
    "target_amount": 210000,
    "years_experience": 7,
    "current_salary": 165000,
    "other_compensation": "Current job: $30k bonus + $80k equity vesting over 4 years. New offer: $40k signing, $200k equity over 4 years, 15% bonus target",
    "specific_concerns": "I really want this job and Im worried they will rescind the offer if I push too hard. Also not sure if my current salary is a good anchor to use.",
    "negotiation_style": "collaborative",
    "reply_email": "candidate@email.com"
  }'
```

**Required:** `job_title`, `offer_amount`, `currency`

---

## Scenarios

| Scenario | When to use |
|---|---|
| `initial_offer` | First offer just received — most negotiation room |
| `counter_received` | You countered and they came back with a revised number |
| `final_round` | Company says this is final — may or may not be true |
| `promotion` | Internal promotion with new comp offer |
| `annual_review` | Annual salary review / raise conversation |
| `competing_offer` | You have another offer and want to use it as leverage |

The scenario shapes the entire coaching approach — a "final offer" situation needs different tactics than an initial offer.

---

## The counter-offer number

Claude gives a specific number, not a range. This is intentional. When you ask for "somewhere between X and Y," employers tend to land at X. Pick a number and defend it.

The number is based on:
- Market data from Tavily search
- Your years of experience
- The gap between current salary and the offer
- Your stated target
- Any competing offers

If the offer is already strong (top of market), Claude will say so and may recommend accepting or negotiating non-salary elements instead.

---

## The exact words to say

The `what_to_say.opening_script` is a call script — something like:

> "I'm really excited about this role and I've done some research on the market. Based on my seven years of experience in this space and the scope of what you've described, I was hoping we could get to $210,000 base. Is that something you have flexibility on?"

The `email_body` is a complete email draft if the scenario is better handled in writing.

These are starting points — read them, adjust them to sound like yourself, then use them.

---

## Pushback responses

Claude generates the 3 most likely pushbacks for your specific scenario and writes out word-for-word responses. For a typical initial offer, these are usually:

1. "That's above our band for this level"
2. "We don't have flexibility on base, but..."
3. "We need an answer by end of week"

Each pushback has a specific response, not general advice about staying calm.

---

## Beyond base salary

When base is genuinely stuck, Claude identifies what else to negotiate. Common items depending on role and company:
- Signing bonus (one-time, doesn't affect base percentage calculations)
- Equity refresh or accelerated vesting
- Additional PTO days
- Remote work flexibility
- Professional development budget
- Performance review timing (getting a review in 6 months instead of 12)
- Title adjustment (sometimes affects future leverage)

Each item comes with a specific ask — not "consider negotiating equity" but "ask for an additional 1,000 RSUs or $25k signing bonus in lieu of base increase."

---

## Competing offers

If you have a competing offer and include it as `competing_offer`, Claude gives specific advice on how and when to use it. This includes:
- Whether to disclose the amount or just confirm you have an offer
- When in the conversation to bring it up
- How to frame it collaboratively rather than as an ultimatum
- When a competing offer actually hurts rather than helps

---

## The "don't do this" section

Every response ends with the single most common mistake for the specific scenario. For an initial offer it's usually: don't negotiate by email if you can call — written counters invite written rejections. For a competing offer scenario it might be: don't bluff about walking away unless you actually will.

---

## Offer type

`offer_type` can be: `annual_salary`, `hourly_rate`, `daily_rate`, `monthly_salary`. The coaching adjusts units accordingly.

---

## Without Tavily

Remove the **Research Market Rate** and **Merge Market Data** nodes, connect **Valid?** directly to **Claude Negotiation Coach**, and replace `{{ $json.marketContext }}` with an empty string. Claude uses its own knowledge of salary ranges — good for well-documented roles (software engineering, product management, finance) but less precise for niche roles.

Alternatively, fill in your own market data in the `market_data_provided` field — paste Glassdoor, Levels.fyi, or LinkedIn Salary data directly.

---

## Limitations

- Market data from Tavily is based on indexed web content which varies in recency and accuracy. Use it as a starting point — verify with role-specific sources (Levels.fyi for tech, Robert Half guides for finance, etc.) before your conversation.
- Claude doesn't know your specific company's compensation philosophy or budget. "Final offer" may genuinely mean final at some companies and be negotiable at others — you know your situation better.
- This is coaching, not legal advice. Employment law (non-competes, offer rescission rights) varies by jurisdiction. Consult a lawyer if you have specific legal questions about your offer.

---

## Ideas

- [ ] Post-negotiation tracker: log what you asked for and what you got — build a personal negotiation history
- [ ] Offer comparison: compare two competing offers across total compensation, growth potential, and lifestyle factors
- [ ] Annual review prep: companion workflow that takes your accomplishments and generates a case for a raise
- [ ] Team use: HR or managers use it to check whether their offers are competitive before extending them

---

## License

MIT.
