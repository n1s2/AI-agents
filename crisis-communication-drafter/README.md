# crisis-communication-drafter

When a crisis hits — an outage, a data breach, a PR scandal, a safety incident — the first 60 minutes matter more than everything that follows. Most organizations spend those 60 minutes in a conference room debating what to say instead of saying something.

This drafts crisis communications for up to five audiences simultaneously: customers, employees, media, and social media. Each gets the right emphasis for their concerns. It includes a holding statement to use immediately while facts are still being gathered, a response timeline with specific actions per timeframe, a list of things not to say, FAQ responses for anticipated questions, and legal review flags for anything that should go through counsel before publishing.

Not a replacement for a crisis PR firm in a serious situation — a starting point that gets something coherent drafted in under 2 minutes.

---

## What it does

1. Accepts a POST: organization name, crisis type, urgency, description, known facts, unknown facts, actions taken, audiences to draft for, spokesperson, legal review flag, tone, claims to avoid
2. Claude drafts across all requested audiences:
   - **Holding statement** — 2–4 sentences for immediate use while facts are gathered
   - **Customer statement** — empathetic, specific to customer impact, actionable
   - **Employee statement** — direct, acknowledges team concerns, emphasizes leadership action
   - **Press statement** — factual, complete, includes spokesperson quote if provided
   - **Social media post** — under 280 characters, acknowledges the situation, points to more
   - **FAQ responses** — pre-drafted answers to likely incoming questions
3. Returns:
   - Situation assessment: severity + reputation risk
   - Communications strategy (overall approach)
   - Response timeline (what to communicate, when, on what channel)
   - What NOT to say (specific phrases to avoid)
   - Legal review flags (statements that need counsel review before publishing)
   - Monitoring recommendations
4. Builds HTML document with all drafts
5. Emails if `reply_email` provided
6. Returns full JSON

---

## Stack

- **n8n** — webhook + workflow
- **Anthropic Claude** (claude-sonnet-4-20250514) — drafting
- **SMTP** — optional email delivery

---

## Setup

### 1. Environment variables

```
FROM_EMAIL=pr@yourcompany.com
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** (optional)

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/draft-crisis-comms \
  -H "Content-Type: application/json" \
  -d '{
    "organization_name": "Meridian Health Tech",
    "org_type": "healthcare SaaS company",
    "crisis_type": "data_breach",
    "urgency": "immediate",
    "crisis_description": "We detected unauthorized access to our production database at 2:47 AM EST. Initial investigation suggests patient health records for approximately 12,000 users may have been accessed. We have contained the breach and isolated the affected systems. We do not yet know what data was specifically accessed or for how long the intrusion was active.",
    "known_facts": "Breach detected 2:47 AM EST today. Systems isolated and breach contained. Approximately 12,000 user records potentially affected. Unauthorized access confirmed. Forensics firm engaged.",
    "unknown_facts": "Exactly what data was accessed. How long intrusion was active. Whether data was exfiltrated or only viewed. Root cause of the vulnerability.",
    "actions_taken": "Affected systems isolated. Forensics firm (SecurePoint) engaged. Internal security team investigating. Legal counsel notified. Board notified at 4 AM. FBI cyber division contacted.",
    "audiences": ["customers", "employees", "media", "regulators"],
    "spokesperson": "Dr. Sandra Osei, CEO",
    "legal_review_required": true,
    "tone": "empathetic_professional",
    "avoid_claims": "Do not state data was not accessed or that no harm occurred — unknown. Do not speculate on cause. Do not name the specific vulnerability.",
    "reply_email": "ceo@meridianhealth.tech"
  }'
```

**Required:** `organization_name`, `crisis_description`, `crisis_type`, `audiences`

---

## Crisis types

`product_outage`, `data_breach`, `pr_scandal`, `safety_incident`, `executive_misconduct`, `financial_issue`, `legal_action`, `service_disruption`, `misinformation`, `natural_disaster`, `employee_incident`, `regulatory_action`, `other`

The type calibrates what Claude emphasizes — a data breach statement has different priorities than a product outage statement.

---

## Urgency levels

| Level | When to use |
|---|---|
| `immediate` | Crisis is public or actively unfolding — need something in minutes |
| `within_hours` | Have some time to gather facts but need drafts ready |
| `within_day` | Situation is contained, preparing for planned announcement |
| `proactive` | Getting ahead of an issue before it becomes public |

---

## The holding statement

The most important output for an immediate crisis. This is what you publish on your status page, send to your team, and read to the first journalist who calls — within 30–60 minutes of a crisis breaking, before you have complete facts.

Good holding statement structure:
1. Acknowledge what happened (what you know)
2. State what you've done or are doing
3. Commit to a specific update time

Claude writes this conservatively — nothing that could create legal liability if the facts change.

---

## The "do not say" list

One of the most useful outputs. Claude generates specific phrases and claims to avoid based on the crisis type and what's still unknown. For a data breach: don't say "no data was compromised" before you know. For a product outage: don't say "all systems are fully restored" if there are lingering issues. For a PR scandal: don't say "this is not who we are" — it's clichéd and ignored.

---

## Legal review flags

When `legal_review_required` is true, Claude flags specific statements in the drafts that should be reviewed by legal counsel before publishing — particularly anything that could be interpreted as admitting liability, making promises that could be held against you, or making factual claims about an ongoing investigation.

For a data breach specifically, HIPAA (US), GDPR (EU), and other privacy regulations have specific notification requirements. This agent helps draft communications but legal review for regulated industries is essential.

---

## FAQ responses

Claude pre-drafts answers to questions your audiences will almost certainly ask. For a data breach, those include: "Was my data accessed?", "What information was exposed?", "What should I do to protect myself?", "How did this happen?" The answers are honest — including honest acknowledgment of what you don't know yet.

---

## Tone options

`empathetic_professional` — warm, responsible, clear (default)
`formal` — more corporate/institutional register
`direct` — facts-forward, minimal emotional framing
`community_focused` — good for nonprofits, local organizations

---

## Regulatory audiences

If `regulators` is in your audiences list, Claude generates a more formal statement appropriate for regulatory bodies — typically more complete, with specific details about actions taken and timeline of discovery. For healthcare (HIPAA), financial services (SEC), or other regulated industries, this needs legal review.

---

## Limitations

- Drafts should be reviewed by your communications team and legal counsel before publishing, especially for serious crises. This tool accelerates drafting — it doesn't replace judgment.
- The output quality depends on what you provide. Unknown facts are acknowledged honestly in the drafts — Claude won't make up facts that aren't provided.
- Crisis communications for publicly traded companies, regulated industries, or situations involving litigation require specialized legal counsel. This agent produces starting points, not final statements.

---

## Ideas

- [ ] Dark site template: generate a pre-built crisis landing page template the team can activate quickly
- [ ] Monitoring brief: connect to the social-media-crisis-monitor agent to auto-trigger when a crisis is detected
- [ ] Post-crisis recap: after resolution, draft the "here's what happened and what we've changed" follow-up communication
- [ ] Stakeholder map: given the crisis type and org, generate a full stakeholder communication plan with sequencing

---

## License

MIT.
