# academic-essay-feedback-agent

Most essay feedback students get is useless. "Needs more analysis." "Argument could be clearer." "Good use of sources." None of that tells you what to actually fix.

This gives feedback the way a good tutor does — pointing to specific sentences, quoting the actual passage that's weak, showing a rewritten version, and being honest about what grade the essay would get and exactly what needs to change to move it up a band.

You paste the essay, tell it the level and type, and optionally paste the assignment prompt. It comes back with a score, a thesis assessment with a suggested rewrite, problem passages identified and fixed, sentence-level revisions, and a priority list of what to fix first.

---

## What it does

1. Accepts a POST: essay text, title, type, academic level, subject, assignment prompt, word count target, specific concerns
2. Claude reads the full essay and returns:
   - Overall score (0–100) and grade equivalent
   - One-line honest verdict
   - Specific strengths with passage references
   - Thesis feedback: current thesis quoted, assessment, stronger rewrite
   - Argument structure assessment, weakest section, one structural suggestion
   - Problem passages: quote → issue → fix
   - Writing quality: problematic sentences with strikethrough original and clean revision
   - Citation feedback if applicable
   - Priority improvement list (numbered, most impactful first)
   - What a top-band version of this essay looks like
3. Formats into a clean HTML feedback report with score bar, color-coded sections
4. Emails to student if `reply_email` provided
5. Returns full JSON in webhook response

---

## Stack

- **n8n** — webhook + workflow
- **Anthropic Claude** (claude-opus-4-5) — essay analysis
- **SMTP** — optional email delivery

---

## Setup

### 1. Environment variables

```
FROM_EMAIL=feedback@yourdomain.com
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** (optional)

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/essay-feedback \
  -H "Content-Type: application/json" \
  -d '{
    "essay_title": "The Role of Propaganda in Consolidating Nazi Power 1933-1939",
    "essay_text": "Adolf Hitler came to power in 1933 and used propaganda...",
    "essay_type": "argumentative",
    "academic_level": "undergraduate",
    "subject": "Modern European History",
    "assignment_prompt": "Assess the role of propaganda in the consolidation of Nazi power between 1933 and 1939.",
    "target_word_count": 2500,
    "specific_concerns": "Not sure if my thesis is specific enough.",
    "citation_style": "Chicago",
    "reply_email": "student@university.edu"
  }'
```

**Required:** `essay_text`, `essay_title`

---

## Academic levels

| Level | What Claude calibrates for |
|---|---|
| `high_school` | Clarity, basic structure, thesis presence, paragraph organization |
| `undergraduate` | Argument quality, evidence use, engagement with sources |
| `graduate` | Sophistication of argument, engagement with scholarship, originality |
| `phd` | Contribution to field, methodological rigor, theoretical framing |
| `professional` | Audience-appropriate register, evidence standards, practical application |

The same essay submitted at different levels gets calibrated feedback — a graduate student's introduction is held to a different standard than a high school one.

---

## Essay types

`argumentative`, `analytical`, `expository`, `comparative`, `research`, `reflective`, `literature_review`, `case_study`, `other`

The type shapes what Claude focuses on. A reflective essay isn't assessed on thesis strength the same way an argumentative one is. A literature review is assessed on coverage and synthesis rather than original argument.

---

## The assignment prompt field

This is the most underused field and makes the biggest difference. If Claude knows what the question was, it can assess whether the essay actually answers it — which is one of the most common reasons essays underperform. Without the prompt, feedback is based on the essay in isolation.

Paste the exact wording from the assignment brief.

---

## The feedback structure

**Score and grade:** Claude scores 0–100 and provides an equivalent grade label calibrated to common grading systems (First / 2:1 / 2:2, or A / B / C, depending on context).

**Strengths:** Genuine strengths with the actual passage quoted — not "good introduction" but "the transition from X to Y in paragraph 3 effectively establishes the central tension."

**Thesis feedback:** Claude quotes the thesis (or notes its absence), assesses it, and writes a stronger version in full. This is often the most immediately useful section.

**Problem passages:** The evidence section quotes specific passages that aren't working, explains the exact problem, and shows the fix. No vague "needs more analysis."

**Sentence revisions:** Weak sentences shown with strikethrough, the problem explained in one line, and a cleaner version written out.

**Priority list:** Numbered from most to least impactful. The goal is to know what to fix first when time is limited.

**Top-band description:** Not "write better" but "this essay would reach distinction if the thesis were narrowed to X, the second body paragraph engaged with Y's counterargument, and the conclusion synthesized rather than restated."

---

## Essay length limits

The webhook accepts up to 10,000 characters — roughly 1,500–2,000 words. For longer essays, submit section by section or trim to the most critical parts you want reviewed.

---

## Building a self-service tool for students

Works well as a self-service writing center tool. Add a Tally form — students paste their essay, select level and type, enter their email. Feedback lands in their inbox without a human reviewer. For a class, batch-process essays from a spreadsheet by calling the webhook in a loop.

---

## Limitations

- 10,000 character limit (~1,500–2,000 words).
- Claude cannot read footnotes or bibliography if not included in the pasted text.
- Subject-specific feedback quality varies — history, philosophy, literature, business well covered; hard science lab reports and technical engineering papers less so.
- The score is Claude's assessment, not an official grade.

---

## Ideas

- [ ] Rubric upload: paste a specific marking rubric, Claude maps feedback to criteria
- [ ] Draft comparison: submit two versions, Claude tracks what improved
- [ ] Cohort mode: batch submissions from a class list
- [ ] Progress tracking: log submissions over time, chart score improvement across drafts

---

## License

MIT.
