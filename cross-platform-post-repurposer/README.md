# FLOOWBOX - Cross-Platform Post Repurposer

Writing the same content five different ways for five platforms is the biggest bottleneck in content marketing. This workflow takes any post and rewrites it natively for Twitter, Instagram, WhatsApp, Threads, and newsletter — each in the platform's actual style.

## What it does

Send any post via webhook. GPT-4o rewrites it for each target platform using that platform's native format — not just reformatting, but genuinely reimagining how the content should be presented. Twitter gets the single sharpest insight in under 280 chars. Instagram gets a hook-first structure with line breaks and hashtags. WhatsApp gets an ultra-casual personal message feel. Newsletter gets a slightly more formal 80-100 word version. All formats save to Notion and preview to Slack.

## Tools Used
- **Orchestration:** n8n
- **Repurposing:** OpenAI GPT-4o
- **Storage:** Notion
- **Preview:** Slack
- **Trigger:** Webhook

## Platform-specific rewrites

| Platform | Format rules | Target length |
|---|---|---|
| Twitter/X | Single sharpest insight, no watering down | Under 280 chars |
| Instagram | Hook + line breaks + 3-5 hashtags | 100-200 words |
| WhatsApp | Personal broadcast, casual, direct | 50-80 words |
| Threads | Same as Twitter, can be 3-post thread | Under 500 chars |
| Newsletter | Slightly formal, CTA to read more | 80-100 words |

## Example LinkedIn → Twitter repurpose

**Original LinkedIn:**
*"After automating 90+ workflows I've noticed a pattern: companies that fail at automation always skip the documentation step. They automate the process before writing down what the process actually is. The result is a workflow nobody understands and everyone fears touching..."*

**Twitter:**
*"The #1 automation failure pattern: they automate before documenting. Nobody knows what the workflow does. Nobody touches it when it breaks. Document first, automate second."*

## Why I built this

One good LinkedIn post was being completely wasted — it would get 200-300 impressions and then disappear. The same insight reformatted for Twitter was reaching a completely different audience. This workflow multiplies every piece of content across all platforms in 30 seconds.

## Setup

1. OpenAI API key
2. Notion integration + Content Calendar DB
3. Slack Bot Token + #content channel
