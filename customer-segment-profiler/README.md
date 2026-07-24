# customer-segment-profiler

Customer personas built from assumptions are useless. "Sarah, 34, Marketing Manager, loves yoga" doesn't help anyone make a product decision. This builds a structured segment profile from actual inputs — observed behaviors, interview insights, usage patterns, churn data, and expansion signals — and outputs something teams can act on: what drives the segment, what messaging resonates, what causes them to leave, and what makes them expand.

---

## What it does

Takes segment name, product, observed behaviors, interview insights, quantitative data, usage patterns, churn data, and expansion signals. Claude builds:

- **One-line description** — who they are and what they need in a sentence
- **Primary job to be done** — the core outcome they hire the product to achieve
- **Profile narrative** — 3–4 sentences describing a real person in this segment, written for colleagues
- **Firmographics** — company size, verticals, geography, team structure
- **Psychographics** — professional goals, fears (relevant to the product), and how they see themselves
- **Buying behavior** — decision trigger, evaluation criteria, decision maker, timeline, budget authority
- **Product relationship** — primary use cases, feature affinity, success definition, expansion triggers, churn risks
- **Messaging** — what resonates, what falls flat, their language (words to reflect back)
- **Segment edges** — which adjacent segment this is most confused with, and the key differentiator
- **Product implications** — specific prioritization decisions this profile suggests
- **Marketing implications** — channel, copy, and positioning implications

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/profile-customer-segment \
  -H "Content-Type: application/json" \
  -d '{
    "segment_name": "Ops-Led SMB",
    "product_name": "Flowdesk",
    "profile_purpose": "product_and_marketing",
    "segment_description": "Small companies (15-60 people) where there is at least one dedicated operations person. Not a tech company — logistics, agencies, professional services.",
    "observed_behaviors": "High frequency of task creation in first 2 weeks then stabilizes. Admin user logs in daily, team members 2-3x per week. Strong correlation between Slack integration setup (within 14 days) and 6-month retention. Rarely use reporting features. Most-used: task assignment, comments, status updates.",
    "interview_insights": "Common phrase: we were drowning in email threads. Most came from spreadsheets (not a competing PM tool). Decision maker is usually the ops manager or COO — IT is not involved. Main success metric they cite: nobody asks me where things stand anymore.",
    "quantitative_data": "Avg ACV $1,800. NPS 52. Median onboarding time to first task: 4 hours. 6-month retention 78%. Expansion common when team grows past 20 users.",
    "product_usage_patterns": "Average session: 8 minutes. Primary feature: task list view (not kanban). Comments used 3x more than attachments. Mobile app used by ~30% of team members but rarely by admin.",
    "churn_data": "Top 3 churn reasons: company too small and outgrew need (10%), switched to Asana when hiring dev team (35%), budget cut (25%). Churn mostly in months 2-4. Rarely churn after month 6.",
    "expansion_data": "Expansion triggers: team grows, adding second department, bringing on client-facing work. Expansion rarely initiated by admin — usually comes after team member says we need more seats.",
    "competitor_context": "Lost to Asana when a dev team joined and pushed for it. Win against Monday.com on simplicity and price.",
    "reply_email": "product@flowdesk.com"
  }'
```

**Required:** `segment_name`, `product_name`

---

## Profile quality vs input richness

The profile is only as good as the inputs. Passing interview insights, behavioral data, and churn signals produces a profile teams can act on. Passing only a segment description produces a structured but generic profile. The `observed_behaviors`, `interview_insights`, and `churn_data` fields have the highest leverage.

---

## Segment edges

The `segment_edges` section identifies the adjacent segment this one is most often confused with and the one thing that clearly separates them. This is useful for targeting and positioning — teams often conflate segments that behave very differently (e.g., "ops-led SMB" vs "founder-led startup") and need a clear differentiator to segment their communications correctly.

---

## Product and marketing implications

Two dedicated sections at the bottom draw out specific, actionable implications from the profile. Product implications: what to build or prioritize for this segment. Marketing implications: which channels to use, how to frame the value proposition, what copy angles to test.

---

## Limitations

- Claude synthesizes the inputs you provide — it cannot access your CRM, product analytics, or interview recordings directly. The richer the inputs you pass, the more accurate and useful the profile.
- Profiles should be validated against actual customers in the segment before being used to drive major decisions.

---

## License

MIT.
