# local-event-newsletter-agent

I used to spend 45 minutes every Friday morning going through Eventbrite and Meetup to figure out what was happening locally. Half the events were garbage. I'd send my friends a list manually, they'd ignore it, someone would discover a good thing on Sunday and we'd all be annoyed we missed it.

This automates the whole thing. Every Friday morning it pulls events from both Eventbrite and Meetup, filters them by the categories you care about, hands everything to Claude which plays editor and writes an actual opinionated newsletter, and sends it to whoever's subscribed.

The key part is the Claude prompt. I spent a while on it. I didn't want summaries — I wanted it to behave like a person who lives in the city and has a point of view. It picks favorites, gives quick takes on others, and has a "skip this" section for events that look bad. It's surprisingly good at this.

---

## What it does

1. Runs every Friday at 8am
2. Fetches upcoming week's events from **Eventbrite** and **Meetup** simultaneously
3. Normalizes both into the same format, filters by your interest categories, deduplicates
4. Sends the list to Claude which writes the newsletter (picks, brief list, skip section)
5. Wraps it in a clean newspaper-style HTML email
6. Gets the subscriber list from Mailchimp
7. Sends the email
8. Logs everything to Google Sheets

---

## Stack

- **n8n** — automation backbone
- **Eventbrite API v3** — event source 1
- **Meetup API** — event source 2
- **Anthropic Claude** (claude-opus-4-5) — editorial writing
- **Mailchimp API** — subscriber management
- **SMTP** — email sending
- **Google Sheets** — send log

---

## Setup

### 1. Get your API keys

**Eventbrite:**  
Create an app at [eventbrite.com/platform/api-keys](https://www.eventbrite.com/platform/api-keys). You want the "Private Token".

**Meetup:**  
Apply for API access at [meetup.com/api](https://www.meetup.com/api). It can take a day or two.

**Anthropic:**  
Get an API key at [console.anthropic.com](https://console.anthropic.com).

### 2. Environment variables

```
EVENTBRITE_API_KEY=your_key
MEETUP_API_KEY=your_key
CITY_NAME=Austin, TX                    # used in Eventbrite location search
CITY_LAT=30.2672                        # used in Meetup lat/lon search
CITY_LON=-97.7431
RADIUS_KM=20                            # search radius
INTEREST_CATEGORIES=music,tech,food,art,outdoor,comedy   # comma-separated
MAILCHIMP_LIST_ID=your_list_id
FROM_EMAIL=newsletter@yourdomain.com
GOOGLE_SHEET_ID=your_sheet_id          # optional, for logging
```

### 3. Credentials in n8n

- **SMTP** — Gmail works with an app password (Google Account → Security → App passwords)
- **Anthropic** — add via LangChain node credentials
- **Mailchimp API** — add via n8n's built-in Mailchimp credential
- **Google Sheets OAuth2** — follow n8n's Google OAuth setup

### 4. Import and activate

Import `workflow.json` via **Workflows → Import from File**. Turn it on. It'll run next Friday at 8am.

To test immediately: open the workflow and click **Execute Workflow** manually.

---

## Customizing the newsletter tone

The whole voice lives in the system prompt of the **Claude Newsletter Editor** node. Some tweaks you might want to make:

- Change the city reference if you want Claude to make local in-jokes or references
- Adjust the three sections (right now: Picks / Also Worth Knowing / Skip This)
- Tell it to include a "free events only" section if that matters to your audience
- Give it a personality name ("Your editor, Marco") if you want it to feel more personal

The prompt is in the node — just double-click it and edit the system message.

---

## Managing subscribers

This workflow uses Mailchimp to get the subscriber list, but sends via SMTP. That means:
- People subscribe through your normal Mailchimp signup form
- The workflow just reads the list and sends directly
- You don't need Mailchimp's campaign tool at all

If you don't want to use Mailchimp, replace the **Get Subscriber List** node with a Google Sheets lookup or just hardcode a few emails in the Send node for testing.

---

## Interest category filtering

The `INTEREST_CATEGORIES` variable controls what gets included. Events are filtered by checking if any of those words appear in the event name, description, or Eventbrite category. Common values:

```
music, tech, food, art, outdoor, comedy, film, yoga, business, gaming, craft, running, networking
```

Start broad (5-6 categories) and narrow it down after seeing a few newsletters. If you're getting too many events, be more specific. If you're getting too few, add more categories or increase the radius.

---

## The "Skip This" section

Claude picks 1-2 events that seem not worth going to. It's usually pretty on the nose — MLM networking events, vague "wellness journeys", things with suspiciously low attendee counts. Occasionally it gets it wrong (it once called a farmers market "aggressively wholesome and probably overpriced" which... fair, but still).

If you don't want this section, remove it from the system prompt.

---

## Google Sheets logging

The **Log to Google Sheets** node appends a row each time the newsletter sends: timestamp, city, number of events found, and the subject line. Useful for tracking over time whether your filters are catching enough events.

Create a sheet called **Send Log** with columns: `date`, `city`, `eventsFound`, `subject`.

---

## Known issues

- Meetup's free API tier has rate limits. If you're running this for multiple cities, add a Wait node between calls.
- Eventbrite sometimes returns events that are technically in the date range but already sold out. Claude usually ignores these but not always.
- The HTML email renders well in Gmail and Apple Mail. Outlook is Outlook.

---

## Ideas I haven't built yet

- [ ] Slack version (just the picks section, no formatting)  
- [ ] Let people reply with preferences that adjust future filters  
- [ ] Multi-city mode (family spread across different cities)  
- [ ] "You went to this last year" deduplication  

---

## License

MIT.
