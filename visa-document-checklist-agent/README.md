# visa-document-checklist-agent

Getting a visa wrong is expensive. Either you show up at the consulate missing a document and have to reschedule, or you submit an incomplete application and wait weeks only to get rejected. The official embassy websites are often confusing, outdated, or buried behind three layers of navigation.

This takes your nationality, destination, and visa type, searches the web for current requirements using Tavily, then Claude synthesizes everything into a structured checklist — organized by category, with each document tagged as always required / usually required / sometimes requested, plus specific practical details (how many copies, validity period, translation requirements, minimum bank balance) and a "watch out" note for the things people commonly get wrong.

It handles edge cases too: previous refusals get specific handling advice, applications involving children get the extra document list, and if the destination is actually visa-free for your passport it says so upfront instead of making you read a whole checklist you don't need.

---

## What it does

1. Accepts a POST: nationality, destination country, visa type, purpose, intended stay, departure date, employment status, whether travelling with children, previous refusals
2. Searches Tavily for current requirements from official and reputable sources
3. Claude synthesizes research + general knowledge into a structured checklist
4. Returns:
   - Visa-free / on-arrival status check upfront
   - Application overview (where to apply, processing time)
   - Full document checklist organized by category, each with requirement level, specific details, and watch-out notes
   - Timeline advice
   - Specific section for previous refusals (if applicable)
   - Additional documents for travelling with children (if applicable)
   - Official sources with links
   - Top tips for this specific nationality/destination combination
5. Emails the formatted HTML checklist if `reply_email` provided
6. Returns full JSON in the webhook response

---

## Stack

- **n8n** — webhook + workflow
- **Tavily API** — real-time visa requirement research
- **Anthropic Claude** (claude-opus-4-5) — checklist generation
- **SMTP** — email delivery (optional)

---

## Setup

### 1. Get Tavily API key

Sign up at [tavily.com](https://tavily.com). Free tier covers typical usage.

### 2. Environment variables

```
TAVILY_API_KEY=tvly-your-key
FROM_EMAIL=visahelper@yourdomain.com
```

### 3. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** (optional)

### 4. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/visa-checklist \
  -H "Content-Type: application/json" \
  -d '{
    "nationality": "Indian",
    "destination_country": "Germany",
    "visa_type": "student",
    "purpose_details": "Masters degree in Computer Science at TU Berlin, starting October 2025",
    "intended_stay_days": 730,
    "departure_date": "2025-09-20",
    "current_residence": "Mumbai, India",
    "employment_status": "student",
    "has_children": false,
    "previous_visa_refusals": false,
    "reply_email": "applicant@email.com"
  }'
```

**Required:** `nationality`, `destination_country`, `visa_type`

---

## Visa types

`tourist`, `business`, `student`, `work`, `digital_nomad`, `family_reunion`, `transit`, `medical`, `retirement`, `investor`

The type shapes the entire checklist — a student visa and a tourist visa for the same country require almost entirely different documents.

---

## Requirement levels

Each document is tagged with one of three levels:

| Level | Meaning |
|---|---|
| **Always required** | You will not be able to apply without this. Applies to every applicant. |
| **Usually required** | Required in the vast majority of cases. Assume you need it unless officially told otherwise. |
| **Sometimes requested** | May be asked for depending on your specific situation, nationality, or the officer processing your application. Have it ready. |

---

## The visa-free check

Before generating the full checklist, Claude checks whether your nationality is actually visa-free or eligible for visa-on-arrival for the destination. If it is, the report leads with that clearly — what it means, for how long, any conditions — so you don't read through a 30-item checklist for a trip that only requires you to have a valid passport.

---

## Previous refusals

If you set `previous_visa_refusals: true`, the checklist includes a dedicated section with specific advice for that situation: how to disclose it (it's almost always required to declare), what documentation helps address the previous refusal, and whether it's worth consulting an immigration lawyer for this particular country/visa type.

This section is important. Previous refusals are not disqualifying for most visas but they must be handled correctly. Failing to declare one when asked is far worse than the refusal itself.

---

## Building a self-service form

Pair this webhook with a Tally.so form and you have a usable self-service tool for travelers, relocation consultants, or HR teams handling international hires. Tally has native webhook support — map the form fields to the body fields and you're done.

---

## Without Tavily

Claude's general knowledge covers visa requirements for most major country-pair combinations, though it may not reflect very recent policy changes. If you don't want to use Tavily:
1. Remove the **Search Visa Requirements** and **Merge Research** nodes
2. In **Claude Checklist Generator**, replace `{{ $json.researchContext }}` with a static note like `"Use your general knowledge only. Note the knowledge cutoff and advise verification."`
3. Connect **Valid?** directly to **Claude Checklist Generator**

The checklist will still be useful but won't reflect web-searched current requirements.

---

## Accuracy and verification

Visa requirements change. Countries update their rules, add new document requirements, or change fee structures without much notice. This tool gives you a strong starting point based on current web research and Claude's knowledge, but:

- Always verify the final list against the official embassy or consulate website for your country
- For complex applications (multi-entry, long-stay, work visas), consider engaging an immigration consultant
- Requirements can differ between processing centers even within the same country

The checklist includes links to official sources which are the authoritative reference. Claude will always say this once in the disclaimer at the top.

---

## Known limitations

- Research quality varies by how well-documented the country pair is online. Major corridors (India→UK, Brazil→USA, Philippines→Canada) have excellent coverage. Less-traveled routes may have thinner research.
- Digital nomad and retirement visa requirements change frequently and vary significantly by country — these checklists should be verified more carefully than tourist or student visas.
- The agent doesn't know your specific personal circumstances beyond what you provide. Edge cases (dual nationality, expired visa in passport, criminal record) are flagged as things to discuss with a consulate or lawyer.

---

## Ideas

- [ ] Track applications: log which checklist was generated and when, follow up with reminders before submission deadline
- [ ] Multi-destination trips: generate checklists for several countries in sequence for a long trip
- [ ] Document expiry reminder: given a passport expiry date, flag if it'll be too close to the travel date for some countries
- [ ] Schengen calculator: for European travel, track days used across multiple trips

---

## License

MIT. Not immigration advice.
