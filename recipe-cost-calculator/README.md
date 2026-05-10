# recipe-cost-calculator

Pricing menu items is one of the things food businesses consistently get wrong, usually because nobody has actually calculated the real cost per serving — they just look at what competitors charge and pick a number.

This takes a recipe with ingredient quantities and costs, calculates the true cost per serving including labor and overhead, and returns multiple pricing options with a specific recommendation. Claude looks at the numbers and gives a direct opinion: which price point makes sense for this type of dish, whether the cost is competitive, and where to trim if needed.

Works for restaurants, caterers, meal kit businesses, food trucks, home bakers pricing orders — anywhere you need to know what a dish actually costs before you decide what to charge.

---

## What it does

1. Accepts a POST: recipe name, ingredients array with quantities and costs, serves count, labor cost/hour, prep time, overhead multiplier, target food cost %
2. Calculates per-ingredient line costs, handles both unit cost and package cost input
3. Computes: batch ingredient cost, labor cost, overhead-adjusted total, cost per serving
4. Generates pricing at: your target food cost %, 3×, 3.5×, and 4× markup
5. Identifies top cost drivers by % of ingredient spend
6. Claude writes a short pricing note: recommended price, competitiveness assessment, one cost reduction suggestion
7. Returns a formatted HTML report with ingredient table, stat cards, pricing options, and advice
8. Emails report if `reply_email` provided

---

## Stack

- **n8n** — webhook + workflow
- **Anthropic Claude** (claude-sonnet-4-20250514) — pricing analysis
- **SMTP** — optional email delivery

---

## Setup

### 1. Environment variables

```
FROM_EMAIL=costing@yourdomain.com
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** (optional)

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/cost-recipe \
  -H "Content-Type: application/json" \
  -d '{
    "recipe_name": "Mushroom Risotto",
    "serves": 4,
    "currency": "USD",
    "target_food_cost_pct": 28,
    "labor_cost_per_hour": 18,
    "prep_time_minutes": 45,
    "overhead_multiplier": 1.15,
    "menu_context": "casual Italian restaurant, mains average $22",
    "ingredients": [
      { "name": "Arborio rice", "quantity": 320, "unit": "g", "package_cost": 4.50, "package_size": 1000 },
      { "name": "Porcini mushrooms (dried)", "quantity": 30, "unit": "g", "package_cost": 8.00, "package_size": 50 },
      { "name": "Cremini mushrooms", "quantity": 400, "unit": "g", "unit_cost": 0.008 },
      { "name": "Parmesan", "quantity": 80, "unit": "g", "package_cost": 9.00, "package_size": 200 },
      { "name": "White wine", "quantity": 120, "unit": "ml", "package_cost": 8.00, "package_size": 750 },
      { "name": "Chicken stock", "quantity": 1200, "unit": "ml", "package_cost": 3.50, "package_size": 1000 },
      { "name": "Shallots", "quantity": 3, "unit": "each", "unit_cost": 0.25 },
      { "name": "Butter", "quantity": 60, "unit": "g", "package_cost": 4.00, "package_size": 500 },
      { "name": "Olive oil", "quantity": 30, "unit": "ml", "package_cost": 12.00, "package_size": 1000 }
    ],
    "reply_email": "chef@restaurant.com"
  }'
```

**Required:** `recipe_name`, `ingredients`, `serves`

---

## Ingredient cost input — two ways

**Option 1: Unit cost** — you know the cost per unit (per gram, per ml, per each):
```json
{ "name": "Eggs", "quantity": 3, "unit": "each", "unit_cost": 0.35 }
```

**Option 2: Package cost** — you know what the pack costs and how big it is:
```json
{ "name": "Arborio rice", "quantity": 320, "unit": "g", "package_cost": 4.50, "package_size": 1000 }
```

The calculator derives unit cost from package cost automatically. You can mix both methods in one recipe.

---

## Labor and overhead

**`labor_cost_per_hour`** — your cook's hourly rate (or your own, if you're the cook). Include superannuation/benefits if applicable.

**`prep_time_minutes`** — active hands-on time for the batch, not passive simmering time. A risotto that takes 45 minutes of stirring is different from a braise that takes 20 minutes of active prep then 3 hours in the oven.

**`overhead_multiplier`** — a multiplier applied to ingredient cost to account for kitchen overhead (utilities, rent allocation, waste, shrinkage). Common values:
- `1.0` — no overhead adjustment (ingredient cost only)
- `1.1` — light overhead (home kitchen, low waste)
- `1.15–1.25` — typical restaurant kitchen
- `1.3+` — high-waste kitchen, expensive premises

---

## Food cost target

`target_food_cost_pct` defaults to 30%. Industry benchmarks:
- Fine dining: 25–30%
- Casual dining: 28–35%
- Fast casual: 25–32%
- Catering: 28–38%
- Food trucks: 30–38%

The "at target food cost %" price is the most useful starting point. The 3×/3.5×/4× options are simpler heuristics some kitchens use when they don't track detailed overhead.

---

## The `menu_context` field

Optional but improves Claude's advice significantly. Tell it: type of venue, what similar dishes sell for, price sensitivity of your customers, whether this is a loss leader or a high-margin item.

Example: `"casual Italian, mains average $22, trying to keep this under $18 to drive orders"`

---

## Interpreting the results

The ingredient breakdown table shows each item's line cost and percentage of total ingredient spend. The top cost drivers are the ones worth examining first when trying to reduce cost:

- Can a premium ingredient be substituted without significantly affecting the dish?
- Is there a cheaper supplier for the biggest line item?
- Can portion size be adjusted without hurting perceived value?
- Is a costly ingredient being used decoratively rather than functionally?

Claude will typically flag the top driver and give a specific suggestion.

---

## Multiple recipes / batch costing

To cost an entire menu, call the webhook once per recipe. If you want a combined report across multiple recipes, collect the responses and build a Google Sheet or Notion page from the results. A simple loop in n8n can call this webhook for each row in a Google Sheet of recipes.

---

## Limitations

- Costs are only as accurate as your input data. If ingredient prices change seasonally or by supplier, update the numbers accordingly — the workflow doesn't pull live commodity prices.
- Waste and trim loss aren't modeled explicitly. A recipe calling for 400g of chicken breast should account for trim — use your actual usable weight, not the raw purchase weight.
- The overhead multiplier is a single blended number. If you want more granular overhead allocation (rent, utilities, packaging each broken out separately), add those fields to the input and sum them in the Calculate Costs node.

---

## Ideas

- [ ] Google Sheets recipe library: store all recipes in a sheet, generate cost reports in bulk
- [ ] Price sensitivity mode: given a target sell price, back-calculate the maximum allowable food cost
- [ ] Seasonal cost tracking: log ingredient costs over time, alert when a key ingredient price spikes
- [ ] Supplier comparison: submit the same ingredient from two suppliers, compare total recipe impact

---

## License

MIT.
