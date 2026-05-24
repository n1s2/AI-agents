# travel-itinerary-builder

Generic travel itineraries are useless. "Day 1: Visit the Eiffel Tower. Day 2: See the Louvre." Anyone with a phone can produce that. What's hard is figuring out the right pace, what to actually prioritize given limited time, what's genuinely worth it vs what's on every list because it was worth it in 1995, and how to group activities so you're not spending half the trip in transit.

This takes your trip details — destination, dates, travel style, interests, dietary needs, must-sees, things to avoid — searches the web for current destination context, then Claude builds a day-by-day itinerary the way a well-travelled friend would. Specific activity recommendations with insider tips, meal suggestions by name or neighborhood, logistics notes, a budget guide, and explicit "worth skipping" advice.

Up to 21 nights. Works for any destination. Deliver as email or JSON.

---

## What it does

1. Accepts a POST: destination, arrival/departure dates, travel style, number of travelers, budget, interests, dietary/mobility needs, must-sees, avoid list, accommodation/flights booked status
2. Searches Tavily for current destination context — seasonal tips, what's on, recent travel advice
3. Claude builds a complete day-by-day itinerary:
   - Trip overview (what makes this destination special right now)
   - Per-day structure: morning / afternoon / evening with activity, specific location, duration, insider tip
   - Meal suggestions per day (specific cafes, restaurants, or neighbourhoods)
   - Logistics notes per day (transport, booking requirements, best timing)
   - 3–5 destination-specific travel tips
   - Getting around guide
   - Budget guide (daily spend estimate per person)
   - Accommodation suggestion (if not already booked)
   - "Worth skipping" list — honest about overrated things
   - Packing note specific to destination and dates
4. Builds a clean day-card HTML email
5. Emails if `reply_email` provided
6. Returns full JSON itinerary

---

## Stack

- **n8n** — webhook + workflow
- **Tavily API** — destination research
- **Anthropic Claude** (claude-sonnet-4-20250514) — itinerary building
- **SMTP** — optional email delivery

---

## Setup

### 1. Environment variables

```
TAVILY_API_KEY=tvly-your-key
FROM_EMAIL=travel@yourdomain.com
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** (optional)

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/build-itinerary \
  -H "Content-Type: application/json" \
  -d '{
    "destination": "Lisbon, Portugal",
    "arrival_date": "2025-09-12",
    "departure_date": "2025-09-19",
    "travel_style": "cultural",
    "travelers": 2,
    "budget": "150 per person per day",
    "currency": "EUR",
    "interests": ["architecture", "food", "fado music", "local markets", "day trips"],
    "dietary_needs": "one person vegetarian",
    "mobility_needs": "steep hills are fine",
    "must_see": "Sintra day trip, Alfama neighborhood, LX Factory",
    "avoid": "overly touristy restaurants near Praça do Comércio",
    "accommodation_booked": false,
    "flights_booked": true,
    "reply_email": "traveler@email.com"
  }'
```

**Required:** `destination`, `arrival_date`, `departure_date`, `travel_style`

---

## Travel styles

| Style | What it produces |
|---|---|
| `budget` | Hostels, free attractions, street food, public transport |
| `mid_range` | 3-star hotels, mix of restaurants, occasional splurge |
| `luxury` | High-end hotels, fine dining, private transfers, experiences |
| `backpacker` | Maximum flexibility, cheap eats, meeting other travelers |
| `family` | Kid-friendly pacing, practical logistics, age-appropriate |
| `romantic` | Intimate settings, sunset timing, special dinners |
| `adventure` | Active experiences, outdoor focus, gear notes |
| `cultural` | Museums, history, local life, arts |
| `foodie` | Meals as the anchor, food markets, cooking classes |
| `slow_travel` | Fewer places, deeper dives, neighborhood feel |

---

## The `interests` field

The more specific the better. Claude uses this to prioritize activities within each day. `["architecture", "fado music", "local markets"]` produces a different Lisbon than `["beaches", "nightlife", "surf"]`.

---

## The `must_see` field

List specific things you've already decided you want to include. Claude builds them into the appropriate day(s) and adds logistics context (book tickets in advance, go first thing in the morning, etc.) rather than just listing them.

---

## The "worth skipping" section

Every itinerary includes 1–3 things Claude recommends avoiding — typically the most over-touristed or over-priced experiences for this travel style. This is the section that tends to be most appreciated. For Lisbon it might be: skip the Tram 28 during peak hours (join a specific walking tour instead), avoid the restaurants right off the main squares, etc.

---

## Day pacing

Claude is explicitly prompted not to pack 8 things per day. A typical full day has 3 time slots (morning, afternoon, evening) with 1–2 activities per slot. For active styles (adventure, backpacker) it'll be denser. For slow travel or family it'll be lighter.

Days are grouped geographically — you won't be sent from one side of the city to the other and back.

---

## Accommodation suggestion

If `accommodation_booked` is `false`, Claude suggests a specific neighbourhood (or 1–2 property names) suited to the travel style and itinerary structure. The suggestion is based on where most of the activities are clustered.

If `accommodation_booked` is `true`, the accommodation field says "already booked" and Claude focuses on integrating logistics with wherever you're staying.

---

## Trip length limits

1–21 nights. For longer trips (month-long travels, multi-country itineraries), split into segments and call the webhook once per destination.

---

## Without Tavily

Remove the **Research Destination** and **Merge Research** nodes, connect **Valid?** directly to **Claude Itinerary Builder**, and replace `{{ $json.destinationContext }}` with an empty string. Claude uses its own knowledge — which is strong for major destinations, thinner for very off-the-beaten-path places.

---

## Limitations

- Prices, opening hours, and specific restaurant availability change. The itinerary gives you the right framework; always verify booking requirements and hours before arriving.
- Claude's knowledge of destinations varies. For major cities (Paris, Tokyo, NYC, Lisbon, Bangkok) it's excellent. For smaller cities and rural areas it's more generic — Tavily research helps fill gaps.
- The itinerary is opinionated. Claude makes choices. If you want every possible option listed, this isn't the right tool — it's designed to make decisions for you, not present a menu.

---

## Ideas

- [ ] Multi-city mode: string multiple destination webhooks together into a single trip document
- [ ] PDF export: render the HTML to a PDF for offline use
- [ ] Google Maps integration: generate a shareable map with all pinned locations from the itinerary
- [ ] Re-plan mode: submit an existing itinerary + a change (weather event, missed flight, spontaneous idea) and get an updated version

---

## License

MIT.
