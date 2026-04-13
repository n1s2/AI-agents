# FLOOWBOX - Cart Abandonment Recovery Email Sequence

70% of online shopping carts are abandoned. A well-timed, personalized recovery sequence converts 5-15% of those back. This workflow runs a 3-email sequence automatically — each email written fresh by GPT-4o using the actual cart contents.

## What it does

Triggered when a cart is abandoned. Runs a 3-email sequence over 72 hours — each email written in real time by GPT-4o using the customer name, cart items, and cart value. Email 1 (1 hour) is a friendly reminder with no pressure. Email 2 (24 hours) adds social proof and mild scarcity. Email 3 (72 hours) offers a 10% discount code expiring in 24 hours as the final push.

## Tools Used
- **Orchestration:** n8n
- **Email Writing:** OpenAI GPT-4o (3 separate writes)
- **Email Delivery:** SMTP
- **Timing:** n8n Wait nodes
- **Trigger:** Webhook (connect to Shopify or WooCommerce)

## 3-email sequence

| Email | Timing | Strategy | Has Discount? |
|---|---|---|---|
| Email 1 | 1 hour | Gentle reminder + cart contents | No |
| Email 2 | 24 hours | Social proof + scarcity | No |
| Email 3 | 72 hours | Final offer + discount code | Yes — 10% |

## Why no discount in Email 1-2

Offering a discount immediately teaches customers to abandon carts intentionally to get discounts. The sequence withholds the discount until the third email — recovering customers who genuinely forgot first, then incentivizing the undecided ones last.

## Why I built this

An e-commerce client was losing ₹2-3L/month in abandoned carts with no recovery system. Installing a basic email tool gave them generic templates. This workflow uses the actual cart contents to write personalized emails every time — reply rates and conversions were significantly higher than their previous generic campaigns.

## Webhook payload (from Shopify/WooCommerce)

```json
{
  "customer_name": "Priya Sharma",
  "email": "priya@example.com",
  "items": [{"name": "Automation Starter Kit", "qty": 1, "price": 2999}],
  "total": 2999,
  "checkout_url": "https://store.com/checkout/recover/xyz",
  "previous_orders": 2
}
```

## Setup

1. Connect Shopify/WooCommerce cart abandonment to webhook URL
2. OpenAI API key
3. SMTP credentials
4. Update sender email and discount code
