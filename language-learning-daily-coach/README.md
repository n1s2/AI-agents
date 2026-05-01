# language-learning-daily-coach

I've started learning languages three or four times and stopped every time. Duolingo gamifies it in a way that feels hollow after a few weeks. Italki tutors are great but expensive and hard to schedule. YouTube channels require me to remember to open YouTube.

What actually helped was getting a lesson email in the morning, already tailored to where I am, what I care about, and how much time I have. I open it with coffee. It takes 15 minutes. I don't have to decide what to study or find the right resource.

This is that. Every morning at 7:30 it emails you a personalized lesson: vocabulary, a grammar point, a short dialogue, a quiz, and a cultural note — all in the language you're learning, at your level, weighted toward your interests and current weak areas.

It tracks what you've covered so it doesn't repeat itself and rotates through different focus areas (vocabulary, grammar, listening, speaking practice) on a set cycle.

---

## What it does

1. Fires every morning at 7:30am
2. Reads your learner profile from Google Sheets (language, level, goal, time available, interests, weaknesses)
3. Reads your progress log to check streak, recent topics, and average quiz scores
4. Determines today's focus based on a rotating schedule across your stated focus areas
5. Sends everything to Claude which generates a complete daily lesson
6. Formats it into a clean reading-friendly HTML email
7. Sends to your inbox
8. Logs the session to keep the history current

---

## Lesson contents

Every lesson includes:
- **Vocabulary** (3–6 words): word, pronunciation guide, translation, example sentence, memory tip
- **Grammar point**: plain-language explanation, 2–3 examples, common mistake at that level
- **Mini dialogue**: a short realistic conversation using today's vocabulary and grammar
- **Quiz**: 3–5 multiple choice questions with hidden answers (click to reveal)
- **Cultural note**: one genuinely interesting fact connected to the content or your interests
- **Streak message**: personalized encouragement based on your current streak
- **Tomorrow preview**: one-line teaser for the next session

---

## Stack

- **n8n** — daily scheduling + workflow
- **Google Sheets** — learner profile + progress log
- **Anthropic Claude** (claude-opus-4-5) — lesson generation
- **SMTP** — email delivery

---

## Setup

### 1. Create the Google Sheet

Two tabs:

**Tab: Profile** — two columns: `setting` and `value`

| setting | value |
|---|---|
| target_language | Japanese |
| native_language | English |
| current_level | A2 |
| learning_goal | travel and everyday conversation |
| daily_minutes | 15 |
| focus_areas | vocabulary,grammar,listening,reading |
| interests | food,anime,hiking,history |
| known_weaknesses | particles,verb conjugation |

**Tab: Progress Log** — columns: `date`, `topic`, `focus`, `language`, `score`, `sent_at`

Leave it empty. The workflow fills it in. The `score` column is for you to manually fill in after each quiz if you want Claude to adapt difficulty over time.

### 2. Environment variables

```
LANG_SHEET_ID=your_google_sheet_id
FROM_EMAIL=lessons@yourdomain.com
LEARNER_EMAIL=you@email.com
```

### 3. Credentials

- **Google Sheets OAuth2** — Google credential setup in n8n
- **Anthropic API** — LangChain node
- **SMTP** — your mail provider

### 4. Import and activate

Import `workflow.json`, activate. Test immediately by running it manually — check that the email lands correctly and the progress log gets a row.

---

## Focus area rotation

The focus areas you list in the profile sheet rotate on a day-of-year cycle. If you put `vocabulary,grammar,listening,reading`, you get:
- Day 1 → vocabulary
- Day 2 → grammar
- Day 3 → listening
- Day 4 → reading
- Day 5 → vocabulary (repeats)

Reorder them in the sheet to prioritize what you want to practice more. Add the same focus twice to get it 2x per cycle.

---

## Levels

Use CEFR levels: `A1`, `A2`, `B1`, `B2`, `C1`, `C2`. Claude adjusts complexity, vocabulary, and grammar concepts accordingly.

Update your level in the Profile sheet whenever you feel you've moved up. There's no automatic progression — you decide when you've leveled up.

---

## Scoring quizzes

The quiz is inside the email — click "Show answer" to reveal each answer. If you want Claude to adapt difficulty over time, after each lesson you can open the Progress Log sheet and fill in a score (0–10) in the `score` column. Claude averages your last 10 scores and uses this to calibrate — lower scores mean simpler content and more review, higher scores mean more challenging material.

If you never fill in scores, it just generates appropriate content for your stated level.

---

## Supported languages

Claude handles all major languages well: Spanish, French, German, Italian, Portuguese, Japanese, Korean, Mandarin, Arabic, Russian, Dutch, Swedish, Polish, and more. For less-common languages (Welsh, Icelandic, Swahili, etc.) the quality is decent but not perfect — worth reviewing before trusting the pronunciation guides.

---

## Sending to multiple learners

The workflow is single-user. For multiple people (a family, a class), duplicate the workflow per learner and point each to a separate profile sheet. The workflows are lightweight so running 10 of them simultaneously is no problem.

---

## Limitations

- No audio. Pronunciation guides are text-only. For languages with tonal or complex phonology (Mandarin, Japanese, Thai) this is a real limitation — use it alongside a pronunciation resource.
- The grammar explanations are good for self-study but won't replace a tutor. If you notice a repeated explanation you disagree with, add it to `known_weaknesses` with a note so Claude addresses it.
- Quiz answers aren't submitted anywhere — there's no automatic score detection. It's on-your-honour self-grading.

---

## Ideas

- [ ] Anki deck export — take vocabulary from each lesson and add it to a spaced repetition deck automatically
- [ ] WhatsApp delivery option
- [ ] Monthly progress summary: streak record, topics covered, quiz score trend
- [ ] Speaking practice prompts (text-based for now, but structured for easy voice extension)

---

## License

MIT.
