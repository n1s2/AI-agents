# FLOOWBOX - Twitter Thread Generator from Blog Post

Writing a blog post and then manually repurposing it into a Twitter thread was taking another hour on top of the writing time. This workflow converts any blog URL into a ready-to-post thread in under 30 seconds.

## What it does

Paste a blog post URL. Jina AI fetches the full article content. GPT-4o converts it into a 10-tweet thread — an attention-grabbing hook, 7 individual insights each under 250 characters, one surprising or controversial takeaway, and a CTA tweet. Every tweet is written to stand alone. The full thread saves to Notion as a draft.

## Tools Used
- **Orchestration:** n8n
- **Content Fetch:** Jina AI reader API
- **Thread Writing:** OpenAI GPT-4o
- **Storage:** Notion content calendar
- **Trigger:** Manual (run when you publish a blog post)

## Thread structure GPT-4o follows

| Tweet | Type | Purpose |
|---|---|---|
| 1 | Hook | Stop the scroll — surprising stat or bold claim |
| 2-8 | Insights | One key point per tweet, standalone value |
| 9 | Hot take | Controversial or counterintuitive angle |
| 10 | CTA | Follow + link back to blog |

## Example hook styles

- "I spent 6 months manually doing what this workflow does in 3 seconds."
- "95% of automation advice is wrong. Here's what actually works:"
- "We 10x'd our client response rate. The tool cost $0."

## Why I built this

Content repurposing was the biggest bottleneck in my content calendar. Blog posts have the research and structure — Twitter threads just need a different format. GPT-4o is remarkably good at extracting the punchiest insights and reformatting them for short-form.

## Setup

1. Jina AI API key
2. OpenAI API key
3. Notion integration + Content Calendar DB
4. Set your blog URL + author handle in the Set node

## Extending this

Connect to an RSS feed trigger to auto-generate a thread every time you publish a new post.
