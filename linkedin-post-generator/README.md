# FLOOWBOX - LinkedIn Post Generator from Blog or URL

Paste any article URL, get 3 ready-to-publish LinkedIn post variations in under 30 seconds.

## What it does

Fetches any article or blog post URL, extracts the content, and uses GPT-4o to generate 3 different LinkedIn post formats — a bold hook post, a storytelling post, and a tips/list post. Each ends with relevant hashtags. You pick whichever format fits best and post directly.

## Tools Used
- **Orchestration:** n8n
- **AI:** OpenAI GPT-4o
- **Content Fetch:** HTTP Request + HTML extraction node
- **Trigger:** Manual (run on demand)

## Flow
```
Manual Trigger
  → Set article URL + tone + audience
  → Fetch article HTML
  → Extract text content
  → GPT-4o generates 3 post variations
  → Clean output
```

## 3 Post Formats Generated

**Post 1 — Bold Hook** (150 words)
Starts with a contrarian or surprising statement. High scroll-stop rate.

**Post 2 — Storytelling** (200 words)
Narrative format. Works well for personal experience and case studies.

**Post 3 — Tips List** (5 bullet points)
Easy to skim. High save rate on LinkedIn.

## Why I built this

Content repurposing for FLOOWBOX — every blog post or case study we write needs to become LinkedIn content. Doing it manually meant 30+ minutes per post. This does all 3 formats in one run.

## How to use

1. Open the workflow in n8n
2. In "Set Input Params" node, change `article_url` to your article
3. Optionally adjust `tone` and `target_audience`
4. Run — pick your favourite post from the output

## Customization

Change the system message in the LLM node to match your personal writing style. The more specific you are about your voice, the better the output.
