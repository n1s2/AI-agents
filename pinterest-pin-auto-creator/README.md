# FLOOWBOX - Pinterest Pin Auto-Creator

Pinterest is a search engine, not a social network — the right keywords drive traffic for months. This workflow turns every blog post into optimized pins automatically every week.

## What it does

Every Tuesday morning, fetches the 5 latest published blog posts from WordPress. GPT-4o writes keyword-rich Pinterest titles and descriptions for each post — optimized for Pinterest search, not just social sharing. Generates image prompt suggestions for each pin. Creates the actual pins via Pinterest API with the right board assignment. Logs everything to Google Sheets and posts a weekly summary to Slack.

## Tools Used
- **Orchestration:** n8n
- **Content Source:** WordPress REST API
- **AI:** OpenAI GPT-4o (SEO-optimized pin content)
- **Publishing:** Pinterest API v5
- **Logging:** Google Sheets
- **Summary:** Slack
- **Schedule:** Weekly Tuesday 9 AM

## Flow

```
Tuesday 9 AM
  → Fetch 5 latest blog posts (WordPress API)
  → Split into individual posts
  → GPT-4o: write pin title + description + keywords
  → Create pin via Pinterest API
  → Log to Google Sheets
  → Post summary to Slack
```

## What GPT-4o generates per pin

```json
{
  "pin_title": "5 AI Automation Workflows Every Founder Needs in 2026",
  "pin_description": "Stop doing repetitive tasks manually. These n8n workflows automate lead capture, invoicing, and client onboarding in minutes...",
  "keywords": ["AI automation", "n8n workflows", "founder productivity", "business automation"],
  "board_suggestion": "Business Productivity and AI Tools",
  "image_prompt": "Vertical infographic showing 5 automation icons on clean white background, bold text headline",
  "best_pin_format": "idea",
  "cta": "Save this for later"
}
```

## Why I built this

Pinterest SEO compounds over time — a well-optimized pin from 6 months ago still drives traffic today. A blog client was creating content consistently but not pinning any of it. Adding this workflow brought them 3,000+ monthly Pinterest visitors within 90 days without any extra manual work.

## Pinterest SEO approach

GPT-4o writes descriptions for search intent — using natural keyword phrases that people actually search, not hashtag spam. Pinterest rewards keyword density in title and first 50 characters of description.

## Setup

1. Pinterest Developer account → App → API access token
2. Pinterest Board ID (from your board URL)
3. WordPress site URL (or swap for any REST API)
4. OpenAI API key
5. Google Sheets ID
6. Slack Bot Token + #content channel
