# FLOOWBOX - Product Description Generator (SEO Optimized)

Writing 5 different versions of a product description for different platforms used to take 2 hours per product. This workflow researches real keywords and generates all 5 versions in under 90 seconds.

## What it does

Send product details — name, category, features, target audience, tone. Perplexity searches for actual SEO keywords with search volume context for that product category. GPT-4o writes five platform-specific descriptions simultaneously, incorporating the researched keywords naturally: long website description, short listing description, Amazon bullet points, meta description, and social media caption. All versions save to Google Sheets.

## Tools Used
- **Orchestration:** n8n
- **Keyword Research:** Perplexity AI Sonar
- **Content Generation:** OpenAI GPT-4o
- **Storage:** Google Sheets
- **Trigger:** Webhook

## 5 outputs per product

| Format | Length | Platform |
|---|---|---|
| Long description | 300-400 words | Website product page |
| Short description | 50-80 words | Category/listing pages |
| Amazon bullets | 5 bullets | Amazon listings |
| Meta description | 150-160 chars | SEO |
| Social caption | Instagram/Facebook length | Social media |

## Example output (partial)

```json
{
  "short_description": "The Automation Starter Kit helps small business owners eliminate 10+ hours of manual work per week. Includes 5 pre-built n8n workflows for invoicing, lead capture, and client onboarding — no coding required.",
  "amazon_bullets": [
    "ELIMINATE MANUAL TASKS: 5 ready-to-use automation workflows included",
    "NO CODING REQUIRED: Built on n8n's visual interface — set up in under 2 hours"
  ],
  "meta_description": "Automate your small business in a weekend. 5 pre-built workflows for invoicing, leads & onboarding. No code needed. Start saving 10hrs/week."
}
```

## Why I built this

An e-commerce client had 200+ products with outdated, generic descriptions. Rewriting all of them manually would take weeks. This processed their entire catalog in a day — and the keyword-researched versions significantly improved their organic search traffic within 3 months.

## Webhook payload

```json
{
  "name": "Automation Starter Kit",
  "category": "Business Software",
  "features": ["5 pre-built workflows", "No code required", "n8n based"],
  "audience": "small business owners",
  "tone": "friendly professional",
  "platform": "shopify"
}
```

## Setup

1. Perplexity API key
2. OpenAI API key
3. Google Sheets ID
