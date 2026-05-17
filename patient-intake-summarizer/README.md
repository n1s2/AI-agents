# patient-intake-summarizer

Providers spend the first few minutes of every appointment re-reading the patient's intake form while the patient sits waiting. A five-page form with free-text fields, medication lists, and symptom descriptions takes time to parse — time that could be spent on the actual conversation.

This takes a patient's intake submission and produces a structured pre-visit brief for the provider: chief complaint in clean clinical language, organized symptom table, medication and allergy summary, relevant history, what the patient specifically wants addressed, documentation gaps, and urgency flags. The provider walks in already oriented.

It's a documentation tool. It does not diagnose. It says so clearly in the output.

---

## What it does

1. Accepts a POST: patient ID, age, sex, chief complaint, symptoms description with duration and severity, current medications, allergies, relevant history, recent labs, patient concerns, urgent flag
2. Claude summarizes into a structured provider brief:
   - 2–3 sentence clinical visit overview
   - Chief complaint formatted in clinical language
   - Symptom table (symptom, duration, severity, character)
   - Active medications list
   - Allergy summary (or NKDA)
   - Relevant history summary
   - Patient concerns (in their own words)
   - Documentation gaps — what's typically collected for this complaint type but wasn't provided
   - Urgency flags — anything that warrants prompt attention
   - Provider review items — documentation prompts, not diagnoses
3. Logs to Google Sheets
4. Returns formatted HTML provider brief + JSON
5. Urgent cases get a red header on the brief

---

## Stack

- **n8n** — webhook + workflow
- **Anthropic Claude** (claude-sonnet-4-20250514) — summarization
- **Google Sheets** — intake log

---

## Setup

### 1. Create the Intakes sheet

One tab: **Intakes** — columns:
```
submitted_at | patient_id | provider | appointment_type | chief_complaint | visit_summary | urgency_flags | documentation_gaps | flag_for_urgent
```

### 2. Environment variables

```
INTAKE_SHEET_ID=your_google_sheet_id
```

### 3. Credentials

- **Google Sheets OAuth2**
- **Anthropic API** (LangChain node)

### 4. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/patient-intake \
  -H "Content-Type: application/json" \
  -d '{
    "patient_id": "PT-20847",
    "patient_age": 54,
    "patient_sex": "Female",
    "provider_name": "Dr. Aisha Okonkwo",
    "appointment_type": "follow-up",
    "chief_complaint": "Worsening knee pain, right side, difficulty going down stairs",
    "symptoms_description": "Pain started about 6 weeks ago after a hiking trip. Initially mild but has gotten progressively worse. Mostly on the inside of the knee. Worse in the morning and after sitting for a long time. Takes about 10-15 minutes to loosen up. No swelling that she can see but it feels puffy sometimes. No locking or giving way.",
    "symptom_duration": "6 weeks",
    "symptom_severity": "5-6 out of 10 at worst, 2-3 at rest",
    "current_medications": "Lisinopril 10mg daily, Atorvastatin 20mg daily, Ibuprofen 400mg as needed (taking most days for the past 2 weeks)",
    "allergies": "Penicillin - rash",
    "relevant_history": "Hypertension diagnosed 2019, hyperlipidemia. No prior knee injuries or surgeries. BMI 28.",
    "recent_labs_or_tests": "Lipid panel 3 months ago - within normal limits. No imaging of the knee.",
    "patient_concerns": "Worried it might be something serious. Wants to know if she needs an MRI. Also concerned about taking ibuprofen long term because of her blood pressure.",
    "flag_for_urgent": false
  }'
```

**Required:** `patient_id`, `chief_complaint`, `provider_name`

---

## Urgency flags

Claude flags anything in the intake that warrants prompt attention — not based on diagnosis but on documentation signals: a patient who marked their pain as 9/10, mentioned chest pain alongside another complaint, noted they've been unable to eat for several days, or submitted a form flagged urgent by staff.

When urgency flags are present, the HTML brief header turns red and the flags appear at the top of the document before anything else.

---

## Documentation gaps

For each chief complaint type, there are standard questions that should typically be collected. If a patient reports knee pain but didn't mention whether there was any trauma, popping sensation, or fever, those absences get flagged as documentation gaps so the provider knows what to ask during the visit rather than realizing mid-appointment.

---

## Patient concerns section

Claude preserves the patient's own words for this section where possible. "She wants to know if she needs an MRI" is more useful than "patient has questions about diagnostic imaging."

---

## What this tool is not

This is explicitly a documentation aid, not a clinical decision support tool. Claude is prompted at `temperature: 0.15` to minimize creative interpretation and stick closely to what was submitted. The output says "Documentation aid only · Not a diagnostic tool" in the footer.

It does not:
- Suggest diagnoses
- Recommend treatments
- Interpret lab values
- Flag drug interactions
- Replace clinical judgment

Anything that looks like a clinical suggestion in the output is a documentation prompt (e.g. "provider may want to verify medication compliance") not a clinical recommendation.

---

## Integrating with intake forms

The webhook accepts a POST, so it pairs with any form that can make an HTTP request:

- **Typeform**: use the webhook integration on form submission
- **Tally**: native webhook support
- **JotForm**: webhook on new submission
- **Custom patient portal**: call the webhook from your form's submit handler

Map the form field values to the webhook body fields. The `patient_id` should come from your patient management system.

---

## HIPAA / data handling notice

Patient intake data is processed through the Anthropic API. Before deploying this in a clinical setting, ensure:

1. You have a Business Associate Agreement (BAA) with Anthropic if required in your jurisdiction
2. Your n8n instance is hosted in a compliant environment
3. The Google Sheets log is appropriately access-controlled
4. You comply with applicable patient privacy regulations (HIPAA in the US, GDPR in the EU, etc.)

This workflow is provided as a technical template. Compliance is your responsibility.

---

## Limitations

- Claude summarizes what's submitted. If a patient's self-reported symptoms are vague or inconsistent, the summary will reflect that vagueness.
- The documentation gaps list is based on common clinical practice patterns, not specialty-specific protocols. For specialized appointments (oncology, psychiatry, etc.), the gaps may not reflect the full picture.
- This tool does not have memory of previous visits. Each intake is processed independently.

---

## Ideas

- [ ] EHR integration: push the structured summary to an EHR system via FHIR API
- [ ] Multi-language support: add a translation step for non-English intake submissions
- [ ] Appointment type templates: different documentation gap lists per appointment type
- [ ] Provider notification: send the brief to the provider's email 30 minutes before the appointment

---

## License

MIT. Not medical advice. Not a diagnostic tool.
