# localization-review-agent

Machine-translated or freelance-translated content often has subtle problems that native speakers on your team would catch immediately but non-speakers can't: awkward phrasing that reveals it's a translation, mistranslated idioms, wrong register (too formal or too casual for the content type), and cultural mismatches. This reviews a source/translation pair for accuracy, tone, cultural fit, and technical issues, then provides a fully corrected version.

---

## What it does

Takes source text, translated text, target language, content type, brand voice notes, character limit, and glossary terms. Claude reviews and produces:

- **Overall quality** — excellent/good/needs_revision/poor with summary
- **Accuracy issues** — specific mistranslations or meaning shifts, with source phrase, current translation, suggested fix, and severity (critical/moderate/minor)
- **Tone and fluency issues** — awkward phrasing, wrong register, unnatural constructions, each with current text, suggested fix, and why
- **Cultural considerations** — idioms, sensitivities, or cultural nuances that need attention
- **Technical issues** — character limit violations, untranslated placeholders, broken formatting
- **Glossary compliance** — whether specified terms were translated correctly and consistently
- **Improved translation** — a fully corrected version incorporating all fixes
- **Ready to ship** flag and reviewer notes

HTML report with source/translation side-by-side, severity-coded issue cards, and the improved translation highlighted.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/review-localization \
  -H "Content-Type: application/json" \
  -d '{
    "source_text": "You are almost there! Just add your first team member to unlock the full power of Flowdesk.",
    "translated_text": "¡Casi estás allí! Solo agrega tu primer miembro de equipo para desbloquear el poder completo de Flowdesk.",
    "source_language": "en",
    "target_language": "es-MX",
    "content_type": "notification",
    "brand_voice_notes": "Friendly and encouraging but not overly casual. We use usted in formal contexts but tú for onboarding since it feels more personal.",
    "context_notes": "This appears in the onboarding flow after account creation, before the user has added any team members.",
    "character_limit": 120,
    "glossary_terms": ["team member", "Flowdesk", "workspace"],
    "cultural_context": "Primary audience is Mexico and other LATAM markets, not Spain",
    "reply_email": "i18n@flowdesk.com"
  }'
```

**Required:** `source_text`, `target_language`, `translated_text`

---

## Content types

`ui_string`, `marketing_copy`, `legal_document`, `email`, `help_docs`, `error_message`, `notification`

Content type calibrates the review. UI strings need to be concise and consistent with existing terminology. Marketing copy can adapt more freely for cultural resonance. Legal documents need precision over naturalness. Error messages need to be clear and non-alarming.

---

## Literal vs meaningful translation

Claude is instructed that literal translation is often wrong translation. A phrase that translates word-for-word but sounds like a translation (rather than natural writing in the target language) gets flagged even if technically "accurate." The improved translation prioritizes how a native speaker would actually write it.

---

## Regional variants

Pass specific locale codes (`es-MX` vs `es-ES`, `pt-BR` vs `pt-PT`) and cultural context notes. Word choice, formality level, and idioms vary significantly across regional variants of the same language.

---

## Glossary compliance

Pass your product's glossary terms (product names, feature names, technical terms that should stay consistent) and Claude checks whether they were translated correctly and consistently, or if they should have been left untranslated (common for product names).

---

## Limitations

- Review quality depends on Claude's proficiency in the target language, which varies by language. For languages with smaller training data representation, treat the review as a helpful first pass, not a substitute for native speaker review.
- This reviews existing translations — it doesn't translate from scratch. For new translations, ask Claude to translate first, then run this review on the output.

---

## License

MIT.
