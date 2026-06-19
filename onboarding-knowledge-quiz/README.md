# onboarding-knowledge-quiz

Most onboarding checks consist of asking a new employee if they've read the handbook. This is not a knowledge check.

This generates role-specific knowledge quizzes from your onboarding materials (or from Claude's knowledge of the topic), sends them to new employees, automatically grades submissions, and emails results back — with explanations for every answer so wrong answers become learning moments rather than just a score.

Two webhooks: one to generate and send a quiz, one to receive and grade answers.

---

## What it does

**Generate quiz (POST `/generate-quiz`):**
- Takes topic, role, company context, optional source content, question count, difficulty, quiz type, passing score
- Claude generates questions calibrated to what actually matters for the role
- Saves quiz to Google Sheets (with the answer key)
- Emails the quiz to the employee (without answers) if `recipient_email` provided
- Returns the quiz JSON with questions stripped of answers

**Submit answers (POST `/submit-quiz`):**
- Takes quiz ID, respondent name, and answers object
- Loads the quiz from Google Sheets to get the answer key
- Grades each question: multiple choice (exact match), true/false (exact match), short answer (any non-empty response scored as attempt)
- Calculates score and pass/fail against configured threshold
- Saves result to Google Sheets
- Emails formatted results with per-question feedback to the employee
- Returns full results JSON

---

## Stack

- **n8n** — two webhooks
- **Google Sheets** — quiz storage + results log
- **Anthropic Claude** (claude-sonnet-4-20250514) — quiz generation
- **SMTP** — quiz delivery + results email

---

## Setup

### 1. Create the Google Sheet

Two tabs:

**Tab: Quizzes** — columns:
```
quiz_id | created_at | topic | role | question_count | passing_score | quiz_json
```

**Tab: Results** — columns:
```
submitted_at | quiz_id | respondent_name | respondent_email | score | correct | total | passed
```

### 2. Environment variables

```
QUIZ_SHEET_ID=your_google_sheet_id
FROM_EMAIL=learning@yourcompany.com
```

### 3. Credentials

- **Google Sheets OAuth2**
- **Anthropic API** (LangChain node)
- **SMTP**

### 4. Import and activate

Import `workflow.json`, activate. Two webhook URLs appear.

---

## Generating a quiz

```bash
curl -X POST https://your-n8n.com/webhook/generate-quiz \
  -H "Content-Type: application/json" \
  -d '{
    "topic": "GDPR and data privacy fundamentals",
    "role": "Customer Success Manager",
    "company_context": "We are a B2B SaaS company processing EU customer data. CSMs handle customer data directly and need to understand what they can and cannot share.",
    "question_count": 10,
    "difficulty": "mixed",
    "quiz_type": "multiple_choice",
    "passing_score": 80,
    "recipient_email": "newemployee@company.com",
    "manager_email": "manager@company.com",
    "source_content": "Key GDPR principles: lawful basis for processing, data minimization, purpose limitation, storage limitation, data subject rights (access, erasure, portability), breach notification within 72 hours, DPA registration requirements."
  }'
```

**Required:** `topic`, `role`

---

## Submitting answers

```bash
curl -X POST https://your-n8n.com/webhook/submit-quiz \
  -H "Content-Type: application/json" \
  -d '{
    "quiz_id": "QZ-1746432891234",
    "respondent_name": "Alex Chen",
    "respondent_email": "alex@company.com",
    "your_answers": {
      "Q1": "B",
      "Q2": "A",
      "Q3": "True",
      "Q4": "C",
      "Q5": "D",
      "Q6": "A",
      "Q7": "B",
      "Q8": "C",
      "Q9": "False",
      "Q10": "A"
    }
  }'
```

The `your_answers` object maps question IDs to the selected answer. For multiple choice, the value is the letter (A/B/C/D). For true/false, it's "True" or "False". For short answer, it's the response text.

---

## Quiz types

| Type | Description |
|---|---|
| `multiple_choice` | 4-option questions with one correct answer |
| `true_false` | Binary questions testing common misconceptions |
| `short_answer` | Open-ended — graded as attempted (any non-empty response) |
| `mixed` | Claude selects the best type per question |

---

## Using source content

The `source_content` field is the most powerful way to get relevant questions. Paste the actual content you want employees to be quizzed on — policy documents, process documentation, product specs, compliance requirements. Claude generates questions directly from that content.

Without `source_content`, Claude generates questions based on standard knowledge for the topic and role. This works well for widely-documented topics (GDPR, security basics, sales methodology) and less well for company-specific processes.

---

## The explanation field

Every question includes an `explanation` field — stored in the answer key (not sent to employees before they answer), shown in the results email. For a wrong answer, it explains why that answer was wrong and what the correct answer means in practice. This turns the quiz from an assessment into a learning tool.

---

## Results email

After grading, the employee receives a formatted email showing:
- Overall score and pass/fail status
- Score by topic area (if multiple topics covered)
- Every question with their answer highlighted green (correct) or red (incorrect)
- The correct answer and explanation for each incorrect response

---

## Short answer scoring

Short answer questions are graded as "attempted" (any non-empty response) rather than scored for content. This makes them suitable for reflection questions or open-ended knowledge prompts where there's no single correct answer. For questions that need substantive grading, stick to multiple choice — fully automated grading of free text is outside the scope of this workflow.

---

## Privacy note

Quiz answers are stored in the Quizzes sheet (answer key) and Results sheet (submissions). Ensure appropriate access controls on the Google Sheet — only quiz creators and HR should see the answer key. Employee scores in the Results tab should be visible to managers.

---

## Limitations

- The quiz is generated once and stored. If your onboarding content changes, regenerate the quiz rather than editing the stored version.
- Short answer grading is pass/fail on submission, not content-graded. For knowledge that requires substantive free-text answers, the quiz is better suited to identifying who completed it than assessing comprehension depth.
- The quiz HTML sent to employees is read-only — it's an informational email showing the questions and quiz ID. To collect answers via a web form, build a simple form that POSTs to `/submit-quiz`. The questions_for_employee field in the JSON response contains all questions without answers for rendering in a form.

---

## Ideas

- [ ] Form integration: build a simple web form from the `questions_for_employee` JSON that submits to `/submit-quiz`
- [ ] Retake policy: allow failed employees to retake after a waiting period
- [ ] Manager notification: when a result is saved, notify the manager with a summary
- [ ] Quiz library: tag quizzes by department and use as a searchable library for all roles

---

## License

MIT.
