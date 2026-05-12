# supplier-quote-comparator

Getting three quotes is standard practice. Actually comparing them properly isn't. Most people sort by price, pick the cheapest, and move on — ignoring payment terms that affect cash flow, delivery windows that might cost more than the price difference, or warranty gaps that become problems six months later.

This takes 2–8 supplier quotes for the same item, normalizes them to a true all-in cost (unit price + delivery), and runs Claude's procurement analysis across the full picture: price, delivery, payment terms, warranty, certifications, red flags in the notes. It returns a clear recommendation with rationale, a runner-up suggestion for when the winner can't deliver, negotiation leverage advice, and a list of missing information to request before committing.

Works for manufacturing components, office supplies, professional services, software subscriptions, raw materials — anything where you have competing quotes.

---

## What it does

1. Accepts a POST: item description, quantity, unit, and an array of quotes (2–8)
2. Normalizes each quote to all-in total (price + delivery cost) and calculates premium vs lowest
3. Sorts quotes by all-in price
4. Sends full comparison to Claude which analyzes: recommendation with confidence level, pros/cons and verdict per quote, red flags, negotiation leverage, missing information, total cost of ownership notes
5. Builds a formatted HTML comparison report with per-quote cards and recommendation section
6. Emails if `reply_email` provided
7. Returns full JSON analysis in webhook response

---

## Stack

- **n8n** — webhook + workflow
- **Anthropic Claude** (claude-sonnet-4-20250514) — procurement analysis
- **SMTP** — optional email delivery

---

## Setup

### 1. Environment variables

```
FROM_EMAIL=procurement@yourdomain.com
```

### 2. Credentials

- **Anthropic API** (LangChain node)
- **SMTP** (optional)

### 3. Import and activate

Import `workflow.json`, activate, grab the webhook URL.

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/compare-quotes \
  -H "Content-Type: application/json" \
  -d '{
    "item_description": "Industrial safety gloves, cut-resistant Level D, size M",
    "quantity": 500,
    "unit": "pairs",
    "currency": "USD",
    "priority_factors": "delivery speed, certification compliance, price",
    "additional_context": "Need for a manufacturing site — EN 388 certification mandatory. Must arrive before June 1.",
    "reply_email": "ops@company.com",
    "quotes": [
      {
        "supplier_name": "SafetyPro Direct",
        "unit_price": 4.20,
        "total_price": 2100,
        "delivery_cost": 0,
        "delivery_days": 7,
        "payment_terms": "Net 30",
        "warranty": "12 months against defects",
        "certifications": "EN 388:2016, ANSI A4",
        "moq": 500,
        "quote_date": "2025-05-05",
        "valid_until": "2025-06-05",
        "notes": "Stock confirmed. Free shipping on orders over $1500."
      },
      {
        "supplier_name": "GlobalWork Supplies",
        "unit_price": 3.85,
        "total_price": 1925,
        "delivery_cost": 145,
        "delivery_days": 14,
        "payment_terms": "Net 60",
        "warranty": "6 months",
        "certifications": "EN 388 (version not specified)",
        "moq": 200,
        "quote_date": "2025-05-04",
        "valid_until": "2025-05-25",
        "notes": "Ships from overseas warehouse. Customs clearance typically 2-3 additional days."
      },
      {
        "supplier_name": "Acme Safety Co.",
        "unit_price": 4.60,
        "total_price": 2300,
        "delivery_cost": 0,
        "delivery_days": 3,
        "payment_terms": "Net 15",
        "warranty": "24 months",
        "certifications": "EN 388:2016, ANSI A4, ISO 9001",
        "moq": 100,
        "quote_date": "2025-05-06",
        "valid_until": "2025-06-06",
        "notes": "Local stock. Can arrange same-week delivery if order placed by Wednesday."
      }
    ]
  }'
```

**Required:** `item_description`, `quotes` (minimum 2)

---

## Quote fields

Each quote object supports:

| Field | Required | Notes |
|---|---|---|
| `supplier_name` | recommended | Defaults to "Supplier N" |
| `unit_price` | one of these | Per unit cost |
| `total_price` | one of these | Unit × quantity |
| `delivery_cost` | optional | Added to total for all-in comparison |
| `delivery_days` | optional | Business days from order to delivery |
| `payment_terms` | optional | Net 30, upfront, etc. |
| `warranty` | optional | Duration and scope |
| `certifications` | optional | Standards met |
| `moq` | optional | Minimum order quantity |
| `quote_date` | optional | When the quote was issued |
| `valid_until` | optional | Quote expiry date |
| `notes` | optional | Free text — Claude reads these carefully |

You don't need to fill every field. Claude adapts analysis to what's provided and flags what's missing.

---

## All-in cost calculation

The workflow automatically computes:
```
all_in_total = total_price + delivery_cost
```

If only `unit_price` is provided, total is calculated as `unit_price × quantity`. All quotes are then sorted by all-in total and each gets a "% premium vs lowest" label. This prevents the common mistake of picking the cheapest unit price while ignoring $200 shipping.

---

## Priority factors

The `priority_factors` field tells Claude what matters most for this purchase. Examples:
- `"price, delivery speed"` — cost-sensitive, time-sensitive
- `"quality, certifications, long-term supplier relationship"` — strategic purchase
- `"delivery reliability, payment terms"` — cash flow matters more than price
- `"lowest total cost"` — pure price play

Claude weighs the recommendation against these priorities.

---

## The negotiation leverage section

Claude always includes a specific piece of negotiation advice: how to use the other quotes as leverage with the recommended supplier. For example: "SafetyPro is $175 cheaper all-in. Use that to ask Acme for a price match or a net-30 terms extension — their 3-day delivery and 24-month warranty justify a small premium but not a 10% one."

This is the field most people skip in a manual comparison and it's often worth more than the price difference.

---

## Missing information

If any quotes are incomplete in ways that affect the decision — certification version not specified, no warranty mentioned, delivery estimate vague — Claude lists what to request before finalizing. You can forward this list directly to the relevant suppliers.

---

## Quote expiry handling

The `valid_until` field is captured and Claude factors it into the analysis. If a quote expires before you'd realistically place the order, it flags this. It's easy to forget that the cheapest option's quote expires in 10 days.

---

## Limitations

- Up to 8 quotes per comparison. More than that and the analysis becomes unwieldy — narrow your shortlist first.
- Claude analyzes based on what you provide. If supplier notes contain important context ("stock limited to 300 units") and you don't include it in the notes field, it won't factor in.
- Payment terms analysis is qualitative. For large orders where cash flow impact is material (e.g. net-60 vs upfront on a $50k purchase), do the actual NPV math separately.
- Certification analysis checks whether certifications are listed but can't verify they're current or applicable to your specific use case. Always confirm with the supplier.

---

## Ideas

- [ ] Historical supplier tracking: log decisions to a sheet, build a supplier performance record over time
- [ ] Reorder trigger: when a supplier is selected, log the decision and set a calendar reminder for reorder based on typical lead time
- [ ] RFQ template generator: given an item description, Claude generates a standardized request-for-quote to send to multiple suppliers
- [ ] Bulk comparison: submit quotes from a spreadsheet row by row, generate a summary table across multiple items

---

## License

MIT.
