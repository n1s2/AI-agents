# FLOOWBOX - Influencer DM Outreach Automator

Generic influencer DMs get ignored. Personalized ones that reference the influencer's actual recent content get replies. This workflow researches each influencer and writes three distinctly different DM variants — you pick the best one and send manually.

## What it does

Send an influencer handle via webhook. Perplexity researches their recent content, audience interests, and previous brand collaborations. GPT-4o writes three personalized DM variants — a short casual one, a value-first pitch, and a story-based connection. Saves all three to an Airtable outreach CRM and posts to Slack for human review. The human always sends the final DM — this never auto-sends.

## Tools Used
- **Orchestration:** n8n
- **Research:** Perplexity AI Sonar
- **DM Writing:** OpenAI GPT-4o
- **CRM:** Airtable
- **Review:** Slack
- **Trigger:** Webhook

## Three DM variants

| Variant | Style | Best for |
|---|---|---|
| Casual | Short, friendly, direct ask | Micro-influencers, casual niches |
| Value-first | Lead with the offer, professional | Mid-tier creators, business niches |
| Story-based | Connect to their specific content | Creators with strong personal brand |

## Example — Value-first variant

```
Hi [Name] — I've been following your automation content for a while and your 
recent thread on replacing manual workflows genuinely helped one of our clients.

We're FLOOWBOX and we build custom n8n automation systems for SMEs. I think 
your audience of founders would find a live walkthrough genuinely useful — not 
just promotional.

Would you be open to a 15-min call to explore what a collaboration might look 
like? No pressure either way.
```

## Why I built this

Writing personalized influencer DMs one at a time took 20-30 minutes per person. This compresses research and writing to under 2 minutes while maintaining the personalization that actually gets replies. The human still chooses and sends — automation handles the work, judgment stays with the person.

## Setup

1. Perplexity API key
2. OpenAI API key
3. Airtable base + Influencer Outreach table
4. Slack Bot Token + #influencer-outreach channel
