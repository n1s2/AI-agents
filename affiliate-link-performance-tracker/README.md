# FLOOWBOX - Affiliate Link Performance Tracker

Affiliate programs die from neglect — top performers leave when they feel unrecognized, inactive affiliates stay on the list doing nothing. This workflow monitors every affiliate's performance weekly, calculates commissions owed, and automatically recognizes top performers.

## What it does

Every Monday, fetches weekly click, conversion, and revenue data from Google Sheets alongside the affiliate roster. Calculates per-affiliate metrics — conversion rate, revenue generated, commission owed, revenue per click. GPT-4o analyzes the full picture: identifying what makes top performers different, flagging underperformers with specific actions, listing inactive affiliates needing outreach, and assessing overall program health. Automatically sends a personal thank-you email to the top performer. Posts a full program health report to Slack.

## Tools Used
- **Orchestration:** n8n
- **Data:** Google Sheets (weekly stats + affiliate list)
- **Metric Calculation:** Code node
- **Analysis:** OpenAI GPT-4o
- **Recognition:** Email (top performer)
- **Report:** Slack
- **Schedule:** Weekly Monday 9 AM

## Metrics calculated per affiliate

| Metric | Formula |
|---|---|
| Conversion Rate | (Conversions / Clicks) × 100 |
| Commission Owed | Revenue × Commission Rate |
| Revenue per Click | Revenue / Clicks |

## Analysis output

```json
{
  "program_health": "healthy",
  "top_performers": [
    {"name": "Rahul M.", "revenue": 12500, "conversion_rate": "4.2%", "what_makes_them_different": "Targets buyers at decision stage with comparison content"}
  ],
  "underperformers": [
    {"name": "Priya S.", "issue": "High clicks, near-zero conversions — traffic quality issue", "suggested_action": "Review their promotional content and audience match"}
  ],
  "affiliates_to_reward": ["Rahul M. — bonus or rate increase to retain"],
  "affiliates_to_offboard": ["test_affiliate — 0 activity in 8 weeks"]
}
```

## Google Sheets schema

**Weekly Stats sheet:**
| Affiliate ID | Clicks | Conversions | Revenue Generated | Week |

**Affiliates sheet:**
| Affiliate ID | Name | Email | Commission % | Join Date |

## Why I built this

Most affiliate programs are completely unmanaged — store owners check the dashboard quarterly at best. This creates weekly visibility into who is performing, who needs help, and who should be recognized — the three things that make affiliate programs actually grow.

## Setup

1. Google Sheets with weekly stats + affiliate roster
2. OpenAI API key
3. SMTP credentials (for top performer email)
4. Slack Bot Token + #affiliate-program channel
