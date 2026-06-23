# social-proof-widget-generator

Social proof copy is small but matters: "12,000+ teams trust us" near a CTA can move conversion meaningfully. Most people either skip it or write something generic. This generates multiple sized variants from your real data points (it won't invent numbers), with placement-specific guidance, in one call.

---

## What it does

Takes a product name, real data points (customer counts, ratings, revenue processed, awards, etc.), and a placement type. Claude writes 1-5 variants calibrated to that placement's space constraints (a hero banner needs more punch than a checkout micro-line), each tagged with character count, which data point it's based on, and why it works. Returns a recommended variant. Builds an HTML preview and emails it if requested.

---

## Stack

n8n, Anthropic Claude (claude-sonnet-4-20250514), SMTP (optional).

---

## Calling the webhook

```bash
curl -X POST https://your-n8n.com/webhook/generate-social-proof \
  -H "Content-Type: application/json" \
  -d '{
    "product_name": "Flowdesk",
    "placement": "pricing_page",
    "tone": "confident",
    "target_audience": "small business owners",
    "variant_count": 3,
    "data_points": [
      "4,200+ small businesses use Flowdesk",
      "4.8/5 average rating from 1,100 reviews on G2",
      "Average customer saves 6 hours/week on scheduling",
      "Featured in Product Hunt Top 5 of the month"
    ],
    "reply_email": "marketing@flowdesk.com"
  }'
```

**Required:** `product_name`, `data_points` (non-empty array)

---

## Placements

`hero_banner`, `pricing_page`, `checkout`, `footer`, `popup`, `email_signature`, `sidebar` — each gets calibrated length and framing guidance in `placement_notes`.

---

## Important: no invented numbers

Claude is explicitly instructed to use only the data points you provide. If you don't have a customer count yet, don't ask for one — pass qualitative data points instead ("Built by engineers who left Stripe and Plaid", "Backed by Y Combinator").

---

## License

MIT.
