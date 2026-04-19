# FLOOWBOX - Local Market Data Collector

National trends tell you what is happening broadly. Local market intelligence tells you what is happening in the city where your customers actually live and work. This workflow monitors the Mumbai/Bangalore startup ecosystem, customer pain points, and networking events every week.

## What it does

Every Monday, three parallel searches gather local market intelligence for the configured city and sector. First stream monitors the local startup ecosystem — funding rounds, new players, accelerator applications. Second stream tracks current pain points and technology adoption signals among the target customer segment. Third stream finds B2B networking events, trade shows, and founder meetups happening this month and next. GPT-4o synthesizes into actionable local market intelligence with specific messaging suggestions and events worth attending.

## Tools Used
- **Orchestration:** n8n
- **Intelligence (x3):** Perplexity AI Sonar (parallel)
- **Synthesis:** OpenAI GPT-4o
- **Storage:** Notion
- **Report:** Slack
- **Schedule:** Weekly Monday 8 AM

## Local intelligence output

```json
{
  "market_pulse": "growing",
  "customer_pain_points_this_month": [
    "GST filing complexity increasing for SMEs after new portal update",
    "Rising operational costs prompting automation interest",
    "Difficulty finding skilled operations staff"
  ],
  "inbound_messaging_suggestions": [
    "Lead with 'Reduce ops costs by 30%' — cost pressure is top of mind this month",
    "GST automation angle is timely given recent portal changes"
  ],
  "events_to_attend": [
    {"event": "Mumbai Founders Meetup", "date": "Apr 25", "why_relevant": "200+ SME founders — direct FLOOWBOX target audience"},
    {"event": "SaaSBoomi Regional", "date": "May 3", "why_relevant": "B2B SaaS founders and buyers in one room"}
  ]
}
```

## Why I built this

FLOOWBOX operates in Mumbai. National SaaS market research was useful but didn't tell me which networking events to attend, what local companies were just funded and might need automation help, or what pain points were top of mind in my specific city this month. This workflow gives that local precision.

## Setup

1. Perplexity API key
2. OpenAI API key
3. Notion integration + DB ID
4. Slack Bot Token + #market-intel channel
5. Update city, sector, target customer in Set Market Config
