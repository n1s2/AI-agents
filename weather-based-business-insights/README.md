# FLOOWBOX - Weather-Based Business Insights Agent

Weather affects business in ways most companies never track — client meeting attendance, ad campaign performance, sales call pick-up rates, and delivery timing all shift with weather conditions. This workflow translates the daily forecast into specific business actions before the workday starts.

## What it does

Every morning at 6 AM, fetches a 3-day weather forecast via WeatherAPI for the configured cities. Perplexity adds context on significant weather events (cyclones, flooding, heatwaves) that might not show in standard forecasts. GPT-4o translates weather conditions into specific business impact — how it affects client availability, whether to reschedule outdoor-adjacent activities, and whether there is a weather-driven opportunity (rainy day = more time at desks = higher email response rates, for instance). High-impact days trigger an urgent alert before the team starts work.

## Tools Used
- **Orchestration:** n8n
- **Weather Data:** WeatherAPI (3-day forecast)
- **Event Context:** Perplexity AI Sonar
- **Business Translation:** OpenAI GPT-4o
- **Alerts:** Slack
- **Schedule:** Daily 6 AM

## Business impact examples

- **Heavy rain in Mumbai:** Client attendance at in-person meetings drops 40%; shift to video calls, increase outbound email
- **Heatwave in Delhi:** Afternoon productivity dips; schedule important calls before 1 PM
- **Normal day:** No changes needed; standard briefing only

## Output

```json
{
  "business_impact_today": "medium",
  "client_activity_forecast": "Lower meeting attendance expected due to rain — clients likely working from home",
  "recommended_actions": ["Move afternoon client visit to video call", "Good day to send email campaigns — higher open rates on work-from-home days"],
  "opportunity": "Remote-work day = clients have more screen time — good for LinkedIn outreach"
}
```

## Why I built this

A Mumbai-based client ran a field sales team. Monsoon season caused wildly inconsistent results — not from the team's effort but from weather affecting client availability. This workflow gave them a daily heads-up to adjust their approach, which smoothed out the monthly revenue variance significantly.

## Setup

1. WeatherAPI key (free tier covers this use case)
2. Perplexity API key
3. OpenAI API key
4. Slack Bot Token + #team and #daily-standup channels
5. Update business type and locations in Set Business Config
