# interview-prep-coach

Job hunting is annoying. One part I particularly hate is the prep — you copy-paste a job description into ChatGPT, get 20 generic behavioral questions, and none of them feel like they're actually specific to the company or role.

This workflow takes a job description, your resume, and the type of interview, and returns a tailored prep pack: real questions that reflect what that company actually tests for, why each question is being asked, what a strong answer looks like, and what traps candidates usually fall into. It also reads your resume against the JD and flags gaps you should be ready to address.

It runs as a webhook so you can call it from anywhere — a simple form, a browser extension, another workflow, whatever.

---

## What it does

1. Accepts a POST request with: job title, company, job description, resume text, interview type, email (optional)
2. Validates the input
3. Sends everything to Claude with a prompt that tells it to act like an experienced interview coach, not a generic question generator
4. Gets back structured JSON: company context, resume gaps, N tailored questions (each with why/strong answer/watch out), and a closing tip
5. Formats it into a clean HTML email
6. If an email was provided, sends the prep pack directly to the user
7. Returns the full JSON in the webhook response
8. Logs the request to Google Sheets

---

## Stack

- **n8n** — webhook + workflow automation
- **Anthropic Claude** (claude-opus-4-5) — question generation and gap analysis
- **SMTP** — email delivery
- **Google Sheets** — request logging (optional)

---

## Setup

### 1. Import the workflow

In n8n: **Workflows → Import from File** → upload `workflow.json`.

### 2. Environment variables

```
FROM_EMAIL=coach@yourdomain.com
GOOGLE_SHEET_ID=your_sheet_id     # optional
```

### 3. Credentials

- **Anthropic API** — add via the LangChain node
- **SMTP** — Gmail with app password works fine
- **Google Sheets OAuth2** — only needed if you want the log

### 4. Get your webhook URL

Activate the workflow. Click the Webhook Trigger node and copy the Production URL. It'll look like:

```
https://your-n8n-instance.com/webhook/interview-prep
```

---

## Calling the webhook

Send a POST request with a JSON body:

```json
{
  "job_title": "Senior Product Manager",
  "company": "Stripe",
  "job_description": "We're looking for a PM to own our checkout product...",
  "resume_text": "5 years at fintech startup, led 3 product launches...",
  "interview_type": "behavioral",
  "email": "you@email.com",
  "num_questions": 8
}
```

**Required fields:** `job_title`, `company`, `job_description`  
**Optional:** `resume_text`, `interview_type`, `email`, `num_questions`

`interview_type` can be: `behavioral`, `technical`, `general` (default: `general`)  
`num_questions` range: 4–15 (default: 8)

---

## Example response

```json
{
  "status": "success",
  "questions_generated": 8,
  "emailed": true,
  "prep_data": {
    "company": "Stripe",
    "role": "Senior Product Manager",
    "interview_type": "behavioral",
    "company_context": "Stripe values extreme ownership and user obsession...",
    "resume_gaps": [
      "No mention of experience with API-first products despite this being core to the role",
      "No metrics cited for the product launches you led"
    ],
    "questions": [
      {
        "id": 1,
        "question": "Tell me about a time you had to make a major product decision with incomplete data.",
        "why_they_ask": "Stripe moves fast and data is never perfect. They want to see your decision-making framework under ambiguity.",
        "strong_answer_looks_like": "A specific situation, the data you had vs. what you wished you had, how you structured the decision, what happened, and what you'd do differently.",
        "watch_out_for": "Saying you 'gathered more data' as the solution — that's not always possible and misses the point."
      }
    ],
    "closing_tip": "Stripe interviewers often ask 'why Stripe specifically' — have a real answer that goes beyond 'payments are interesting'. Know one product decision they made in the last year that you have an opinion on."
  }
}
```

---

## Building a form on top of this

The webhook is just a POST endpoint, so you can build a simple HTML form that hits it. A minimal example:

```html
<form id="prep-form">
  <input name="job_title" placeholder="Job title" required>
  <input name="company" placeholder="Company" required>
  <textarea name="job_description" placeholder="Paste job description..." required></textarea>
  <textarea name="resume_text" placeholder="Paste your resume (optional)"></textarea>
  <select name="interview_type">
    <option value="general">General</option>
    <option value="behavioral">Behavioral</option>
    <option value="technical">Technical</option>
  </select>
  <input name="email" type="email" placeholder="Email (to receive prep pack)">
  <button type="submit">Generate Prep Pack</button>
</form>

<script>
document.getElementById('prep-form').addEventListener('submit', async (e) => {
  e.preventDefault();
  const data = Object.fromEntries(new FormData(e.target));
  const res = await fetch('YOUR_WEBHOOK_URL', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  });
  const result = await res.json();
  console.log(result);
});
</script>
```

---

## Customizing the question style

Everything is in the Claude prompt inside the **Claude Question Generator** node. A few things worth tweaking:

- Change the number of sections (right now: why they ask / strong answer / watch out for)
- Add a "sample STAR structure" field if you want Claude to sketch out an actual answer template
- Change the output schema if you're rendering this somewhere custom
- Adjust tone — the default is direct and slightly blunt. If you want warmer, change the system prompt.

---

## Privacy note

Job descriptions and resume text are sent to Anthropic's API. Don't paste resumes with SSNs, passport numbers, or anything sensitive. The workflow doesn't store resume text anywhere — it passes through in memory only. The Google Sheets log only captures metadata (company, role, question count, timestamp).

---

## Known limitations

- Works best for corporate / tech / startup roles. Less useful for trades, creative roles, academic positions — Claude's context on those interview formats is thinner.
- "Technical" interview type improves the questions somewhat but Claude doesn't generate actual coding problems. For that you'd want a separate flow with LeetCode/HackerRank integration.
- The gap analysis is only as good as the resume text you provide. If you paste a bad resume excerpt, you get vague gaps.

---

## Ideas for later

- [ ] Multi-round mode (generate different questions for phone screen vs. onsite)  
- [ ] Answer practice: submit your answer, Claude scores it and gives feedback  
- [ ] Glassdoor scraping to supplement with real reported interview questions  
- [ ] Notion or Obsidian export  

---

## License

MIT.
