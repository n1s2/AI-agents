# FLOOWBOX - Facebook Ad Copy Generator

Writing and testing multiple ad copy variants is how winning campaigns are built. This workflow generates 5 tested-framework variants from a simple brief — ready to A/B test immediately.

## What it does

Send a POST request with your product, target audience, core pain point, offer, and CTA. GPT-4o writes 5 complete Facebook ad variants — one each in PAS, AIDA, Social Proof, FOMO, and Story frameworks. Each variant includes the primary text body, a headline under 40 characters, and a description under 30 characters. All five save to Google Sheets. The top recommended variant and test order go to Slack.

## Tools Used
- **Orchestration:** n8n
- **AI Copywriting:** OpenAI GPT-4o
- **Storage:** Google Sheets (ad copy library)
- **Notification:** Slack
- **Trigger:** Webhook

## 5 frameworks generated

| Framework | Structure | Best For |
|---|---|---|
| PAS | Problem → Agitate → Solution | Cold audiences |
| AIDA | Attention → Interest → Desire → Action | Warm audiences |
| Social Proof | Results first → then offer | Retargeting |
| FOMO | Urgency + scarcity | Limited offers |
| Story | 3-sentence micro-narrative | Brand awareness |

## Example output — PAS variant

```json
{
  "framework": "PAS",
  "primary_text": "Still copying leads manually from WhatsApp into your CRM? Every hour you spend on that is an hour not spent closing deals. FLOOWBOX automates the entire process — WhatsApp message arrives, lead qualifies, CRM updates, reply sends. All in 3 seconds.",
  "headline": "Stop copy-pasting leads",
  "description": "Automate in one afternoon",
  "hook_strength": "strong",
  "best_for": "cold audience"
}
```

## Webhook payload

```json
{
  "product": "FLOOWBOX automation service",
  "audience": "SaaS founders with 5-50 person teams",
  "pain_point": "spending hours on repetitive manual tasks",
  "offer": "free automation audit call",
  "cta": "Book Free Call",
  "objective": "conversions"
}
```

## Why I built this

Ad creative testing was bottlenecked by copywriting time. A client needed 5 variants to test before launch — manually writing all 5 took half a day. This generates all 5 in 40 seconds, covering every major copywriting framework so the A/B test is structured, not random.

## Setup

1. OpenAI API key
2. Google Sheets ID (ad copy library)
3. Slack Bot Token + #marketing channel
4. POST your brief to the webhook URL
