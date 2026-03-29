# FLOOWBOX - Instagram Caption Generator (GPT-4o Vision)

Writing captions for every post was taking 20-30 minutes per image. This workflow looks at the image itself and writes 3 ready-to-use caption variants in seconds.

## What it does

Send any image URL to the webhook along with brand voice, target audience, and CTA preferences. GPT-4o Vision analyzes the image visually and writes three caption variants — an engaging hook-style caption, a storytelling narrative, and a minimal clean version. Each comes with relevant hashtags. All three options save to a Notion content calendar as a draft, and the webhook responds with the captions immediately.

## Tools Used
- **Orchestration:** n8n
- **AI Vision:** OpenAI GPT-4o Vision (direct API call)
- **Storage:** Notion content calendar
- **Trigger:** Webhook

## Caption styles generated

| Style | Description | Best for |
|---|---|---|
| Engaging | Hook opener + emojis + hashtags | High reach posts |
| Storytelling | Narrative format, 3-4 sentences | Brand building |
| Minimal | 1-2 lines, clean, no fluff | Product shots |

## Example output

```json
{
  "captions": [
    {
      "style": "engaging",
      "text": "This is what 3 months of building looks like 🚀 We started with zero clients...",
      "hashtags": ["#buildinpublic", "#startuplife", "#automation"]
    },
    {
      "style": "storytelling",
      "text": "Six months ago I was doing everything manually. Now FLOOWBOX handles 80% of our ops automatically.",
      "hashtags": ["#founder", "#AI"]
    },
    {
      "style": "minimal",
      "text": "Automate the repetitive. Focus on what matters.",
      "hashtags": ["#automation", "#productivity"]
    }
  ],
  "image_description": "Person working at laptop in coffee shop, warm lighting",
  "best_posting_time": "Tuesday or Wednesday 7-9 AM"
}
```

## Why I built this

Content clients were spending more time writing captions than creating content. GPT-4o Vision reads the actual image — so captions are contextually relevant, not generic. Three variants means the human still picks, but without starting from a blank page.

## Webhook payload

```json
{
  "image_url": "https://...",
  "brand_voice": "professional, witty",
  "topic": "product launch",
  "audience": "SaaS founders",
  "cta": "Link in bio",
  "hashtags": true
}
```

## Setup

1. OpenAI API key (GPT-4o with Vision)
2. Notion integration + Content Calendar DB ID
3. Webhook URL configured in your upload tool
