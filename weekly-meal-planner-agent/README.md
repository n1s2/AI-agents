# weekly-meal-planner-agent

My partner and I used to spend 20 minutes every Sunday having the same conversation: "what do you want to eat this week?" "I don't know, what do you want?" Nobody ever had an answer and we'd default to the same 4 meals on rotation.

I built this to make that conversation unnecessary. Every Saturday morning an email arrives with a full dinner plan for the week, a grocery list sorted by category, and ingredient overlap notes so we're not buying a whole bunch of parsley for one dish.

It reads your preferences and dietary restrictions from a Google Sheet and — importantly — it tracks what you've eaten recently so it doesn't suggest the same meals on repeat.

---

## What it does

1. Fires every Saturday at 9am
2. Reads your household preferences from a Google Sheet (people, dietary restrictions, cuisines, time constraints, budget)
3. Reads the last 14 days of meal history from the same sheet to avoid repetition
4. Sends both to Claude which plans 7 dinners tailored to your constraints
5. Claude also generates a categorized grocery list and notes ingredient overlaps across the week
6. Sends you a formatted HTML email with the full plan, expandable recipe details, and a printable grocery list
7. Saves this week's meals back to the history sheet

---

## Stack

- **n8n** — workflow and scheduling
- **Google Sheets** — preferences storage + meal history
- **Anthropic Claude** (claude-opus-4-5) — meal planning and grocery consolidation
- **SMTP** — email delivery

---

## Setup

### 1. Create the Google Sheet

Create a sheet with three tabs:

**Tab 1: Preferences**
Two columns: `setting` and `value`. Add rows for:

| setting | value |
|---|---|
| household_size | 2 |
| dietary_restrictions | vegetarian |
| disliked_ingredients | cilantro, olives |
| cuisine_preferences | Italian, Japanese, Mexican |
| weekday_cooking_time | 30 minutes |
| weekend_cooking_time | 60 minutes |
| weekly_budget | moderate |
| include_breakfast | no |
| include_lunch | no |

**Tab 2: Meal History**
Three columns: `date`, `meal_name`, `week_of`. Leave it empty — the workflow fills it.

**Tab 3: (optional) Pantry**
Not used yet but a good place to track staples you always have.

### 2. Environment variables

```
MEAL_PLANNER_SHEET_ID=your_google_sheet_id
FROM_EMAIL=you@domain.com
USER_EMAIL=you@domain.com
```

### 3. Credentials

- **Google Sheets OAuth2** — follow n8n's Google setup guide
- **Anthropic API** — add to the LangChain node
- **SMTP** — Gmail with app password works

### 4. Import and activate

Import `workflow.json`, activate, and it'll run next Saturday. To test: open the workflow and click Execute Workflow manually.

---

## What the email looks like

Each day gets a card with the meal name, a one-liner description, and total cook time. Click "Ingredients + steps" to expand the recipe. Below the 7 meal cards is a weekend prep tip and then the full grocery list sorted into: produce, protein, dairy, pantry, other.

It renders well in Gmail, Apple Mail, and most email clients. I haven't tested Outlook.

---

## Dietary restrictions

Claude handles common restrictions well: vegetarian, vegan, gluten-free, dairy-free, halal, kosher, nut-free. For more specific medical diets (low-FODMAP, renal diet) it's hit or miss — you'd want to review the output more carefully.

---

## Budget levels

The `weekly_budget` setting accepts plain English: "tight", "moderate", "comfortable", "no limit". Claude interprets these sensibly. If you want to be specific, "under $100 for 2 people" also works.

---

## Cuisine preferences

The more specific the better. "Asian" is vague. "Japanese, Thai, Vietnamese" is better. Claude uses this as inspiration, not a strict filter — if you list 3 cuisines you won't get one of each per week, but the meals will lean toward those flavors.

---

## Adding breakfast/lunch

Set `include_breakfast` and/or `include_lunch` to `yes` in the Preferences sheet. The email and grocery list expand accordingly. Breakfast suggestions are weekdays only (Saturday and Sunday are assumed to be flexible).

---

## Meal history and repetition

The workflow checks the last 14 days of the history sheet and tells Claude to avoid those meals. This prevents the "third time this month I've planned chicken stir fry" problem. If you want a longer memory, change the `cutoff` days in the **Merge Preferences** node.

---

## Running for multiple households

The workflow is set up for one household. If you want to run it for multiple people (family members in different cities, a shared house with different schedules), you'd need to:
1. Add a `user_id` column to the Preferences and History sheets
2. Split the workflow into a loop per user
3. Or just duplicate the workflow per person — it's simple enough

---

## Known issues

- If your Google Sheet has empty rows or differently-named columns, the preference parsing can be flaky. The column names need to match exactly: `setting` and `value`.
- Claude occasionally suggests meals that technically violate a restriction (e.g., "add parmesan" for a dairy-free plan). Always scan before shopping.
- The grocery list consolidation is pretty good but not perfect — it won't know you already have a full bottle of olive oil.

---

## What I'd build next

- [ ] Pantry awareness (tell it what you already have, subtract from grocery list)
- [ ] WhatsApp version — send the grocery list to a shared chat
- [ ] Per-meal feedback (rate meals 1-5 via a quick form, factor into future planning)
- [ ] Seasonal ingredient awareness based on location and time of year

---

## License

MIT.
