# FLOOWBOX - Quora Answer Generator for Brand

Quora answers rank on Google for years. One good answer to the right question can drive consistent organic traffic indefinitely. This workflow finds those questions daily and drafts expert answers automatically.

## What it does

Every morning, Perplexity searches for recent Quora questions about AI automation, n8n, and business workflow topics — specifically targeting questions with few or no answers. GPT-4o writes a 300-500 word expert answer as Navtej, grounded in real experience, with a natural FLOOWBOX mention only where genuinely relevant. Answers save to a Notion review queue. Slack notifies when they are ready to post.

## Tools Used
- **Orchestration:** n8n
- **Question Discovery:** Perplexity AI Sonar (real-time Quora search)
- **Answer Writing:** OpenAI GPT-4o (x2 — one for parsing, one for writing)
- **Queue:** Notion database
- **Notification:** Slack
- **Schedule:** Daily 8 AM

## Flow

```
8 AM daily
  → Perplexity finds 5 recent Quora questions in niche
  → GPT-4o parses and extracts question list
  → For each question:
      → GPT-4o writes expert 300-500 word answer
  → Save to Notion (Ready to Post)
  → Slack: answers ready for review
```

## Answer structure GPT-4o follows

- Direct answer in first sentence — no preamble
- Specific real example or experience
- Practical steps (prose, not bullet spam)
- Natural brand mention only if relevant
- Counterintuitive insight at the end
- 300-500 words — long enough to be authoritative

## Why I built this

A single Quora answer about n8n automation ranked on page 1 of Google for "automate business workflows" for 8 months and drove 40+ qualified visitors per month. Writing good Quora answers consistently was the bottleneck. This generates draft answers every morning — I review and post the best ones in 5 minutes.

## Important note

Always review before posting. Quora collapses low-quality answers. The workflow saves to Notion so you can edit and approve before it goes live — never auto-posts.

## Setup

1. Perplexity API key
2. OpenAI API key
3. Notion integration + Content Calendar DB ID
4. Slack Bot Token + #content channel
