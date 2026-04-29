# sleep-debt-recovery-coach

Honestly, I built this because I kept waking up tired no matter how many hours I slept. I had an Oura ring collecting data I wasn't actually reading, and I figured — why not let an AI look at it and tell me what's wrong.

It runs every morning at 7am, pulls your last 7 days of Oura sleep data, calculates how much sleep debt you've actually accumulated, and sends you a short personalized report to your inbox. If your debt is really bad (10+ hours), it also pings you on Slack so you can't ignore it.

The coaching part uses Claude. I specifically told it to write like a knowledgeable friend, not a wellness app. Less "optimize your sleep hygiene" and more "you're averaging 5.8h, that's not fine, here's what to do this week."

---

## What it does

- Pulls sleep data from Oura Ring API (last 7 days)
- Calculates nightly sleep debt against an 8h baseline
- Identifies your best and worst night, average efficiency, REM/deep sleep
- Sends that to Claude which writes a ~300 word coaching note
- Formats it into a clean HTML email with key stats at the top
- Emails it to you every morning
- Optional: Slack alert if debt exceeds 10 hours

---

## Stack

- **n8n** — workflow automation
- **Oura Ring API v2** — sleep data source
- **Anthropic Claude** (claude-opus-4-5) — coaching analysis
- **SMTP** — email delivery
- **Slack** (optional) — urgent alerts

---

## Setup

### 1. Oura API token

Go to [cloud.ouraring.com/personal-access-tokens](https://cloud.ouraring.com/personal-access-tokens) and generate a personal access token. You don't need OAuth unless you're building this for multiple users.

### 2. Environment variables

Set these in your n8n instance:

```
OURA_API_BASE=https://api.ouraring.com
OURA_ACCESS_TOKEN=your_token_here
FROM_EMAIL=you@yourdomain.com
USER_EMAIL=you@yourdomain.com
SLACK_CHANNEL=#sleep-alerts   (optional)
```

### 3. Credentials

- Add an **SMTP credential** in n8n (Gmail works fine with an app password)
- Add an **Anthropic API credential** under the LangChain node
- Add a **Slack credential** if you want the urgent alert

### 4. Import the workflow

In n8n, go to **Workflows → Import from File** and upload `workflow.json`. Activate it and it'll run the next morning at 7am.

---

## Customizing the sleep target

The default baseline is 8 hours. If you're someone who genuinely functions well on 7h (some people do), change this line in the **Calculate Sleep Debt** node:

```js
const RECOMMENDED_SLEEP_SECONDS = 7 * 3600;  // change 7 to whatever works for you
```

---

## Customizing the trigger time

The Schedule Trigger is set to 7:00am. Change the `triggerAtHour` value in the node to whatever time you want — just make sure it's after Oura has synced your previous night's data (usually takes 1–2 hours after waking).

---

## What the email looks like

Three stats at the top (sleep debt, avg per night, days tracked), then a coaching note below it. No charts, no dashboards — just text that's actually useful. I wanted something I'd actually read, not something I'd swipe away.

---

## Limitations

- Only works with Oura Ring. If you use Garmin, Whoop, or Apple Watch, you'd need to swap out the API call — the rest of the workflow is the same.
- Claude's advice is not medical advice. If you have chronic sleep issues, see a doctor.
- If Oura doesn't have data for a day (e.g. you didn't wear the ring), that night is skipped rather than counted as zero sleep.

---

## Roadmap / things I might add

- [ ] Google Sheets logging so I can track debt over time
- [ ] A weekly summary in addition to the daily one
- [ ] Correlation between sleep score and next-day productivity (need to pull from a second source for that)
- [ ] Garmin Connect API version

---

## License

MIT. Do whatever you want with it.
