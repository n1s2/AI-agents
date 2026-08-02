# hiring-scorecard-generator

A hiring debrief where everyone just says "I liked them" or "I'm not sure" wastes the interview panel's collective signal. This synthesizes feedback from multiple interviewers into a structured scorecard: where interviewers agree, where they diverge, evidence-backed criteria assessment, and specific things to discuss in the debrief — without hiding disagreement or fabricating evidence that isn't in the notes.

---

## What it does

Takes candidate name, role, seniority, must-have/nice-to-have criteria, and feedback from up to 10 interviewers (each with recommendation, competencies assessed, and detailed notes). Claude produces:

- **Overall recommendation** — strong_hire/hire/lean_hire/lean_no_hire/no_hire with confidence level (high/medium/low)
- **Synthesis summary** — overall picture, where interviewers agree, where they diverge
- **Criteria assessment** — each criterion with evidence for and against pulled directly from interviewer notes, and verdict (met/partially_met/not_met/insufficient_evidence)
- **Interviewer agreement** — topics where there's strong agreement, some agreement, or disagreement
- **Disagreements to discuss** — specific topics where interviewers differ, who said what, and how to resolve it in debrief
- **Strengths and concerns** — evidence-backed, not generic
- **Reference check questions** — specific things to verify based on interview signals
- **Debrief discussion points** — what the hiring team should discuss before deciding

HTML scorecard with recommendation badge, criteria cards showing evidence for/against, and disagreement cards highlighted in red.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-hiring-scorecard \
  -H "Content-Type: application/json" \
  -d '{
    "candidate_name": "Alex Chen",
    "role_title": "Senior Backend Engineer",
    "seniority_level": "senior",
    "must_have_criteria": [
      "5+ years backend engineering experience",
      "Experience building and maintaining production integrations",
      "Strong system design skills"
    ],
    "nice_to_have_criteria": [
      "Experience with Node.js specifically",
      "Startup experience"
    ],
    "resume_summary": "7 years experience, last 3 at a Series B startup building integrations, previously at a larger company doing backend infrastructure.",
    "reply_email": "priya@flowdesk.com",
    "interviewer_feedback": [
      {
        "interviewer": "Priya Sharma",
        "interview_type": "system_design",
        "recommendation": "strong_hire",
        "competencies_assessed": ["system design", "technical communication"],
        "notes": "Excellent system design interview. Walked through designing a rate-limited API gateway with clear tradeoffs. Asked great clarifying questions before diving in. Communicated complex ideas simply. This is exactly the caliber we need for the integrations work."
      },
      {
        "interviewer": "Tom Walsh",
        "interview_type": "technical_pairing",
        "recommendation": "hire",
        "competencies_assessed": ["coding", "debugging"],
        "notes": "Solid coding skills, worked through the pairing exercise methodically. Took a bit longer than expected on the edge case handling but got there. Good communication throughout, explained reasoning as they went. Some hesitation with async patterns in Node specifically, but fundamentals were strong."
      },
      {
        "interviewer": "Sara Kim",
        "interview_type": "behavioral",
        "recommendation": "lean_hire",
        "competencies_assessed": ["collaboration", "ownership"],
        "notes": "Good stories about past project ownership. One story about a production incident was strong - clear ownership and postmortem process. However when asked about working with product on scope changes, answer felt a bit rehearsed and didn't have much specificity. Might just be interview nerves."
      }
    ]
  }'
```

**Required:** `role_title`, `interviewer_feedback`

---

## Evidence-based, not fabricated

Claude is explicitly instructed to never fabricate evidence not present in interviewer notes. If a criterion can't be assessed from the feedback provided, the verdict is `insufficient_evidence` rather than a guessed rating. This keeps the scorecard honest and useful.

---

## Disagreement surfaced, not hidden

When interviewers give different signals (e.g., one says strong_hire, another says lean_hire), the scorecard surfaces this explicitly with the specific topic of disagreement and a suggested way to resolve it in the debrief — rather than averaging it into a mushy "hire" recommendation that obscures the real conversation needed.

---

## Reference check questions

Based on signals in the interview notes (like Sara's note above about the "rehearsed" answer on scope changes), Claude suggests specific reference check questions to verify or dig deeper on things the interview process couldn't fully resolve.

---

## Limitations

- Synthesis quality depends on interviewer note quality. "Good candidate, hire" as notes produces a thin scorecard. Detailed notes with specific examples produce a scorecard with real evidence.
- This synthesizes feedback — it doesn't replace the hiring team's judgment or the debrief conversation itself.

---

## License

MIT.
