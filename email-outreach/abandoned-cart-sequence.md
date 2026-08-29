---
name: abandoned-cart-sequence
owner: cartlift
category: Email & outreach
description: Drafts a three-mail abandoned-cart sequence from the cart contents and the store's voice — useful, specific, and honest about discounts.
version: v2
license: MIT
updated: 2026-08-17
recommended: false
security_checked: true
url: https://emdly.com/skills/cartlift/abandoned-cart-sequence
raw: https://emdly.com/raw/cartlift/abandoned-cart-sequence.md
install: npx @emdly/cli add cartlift/abandoned-cart-sequence
---

# Abandoned cart sequence

Three mails, each with a different job: remind, help, decide. None of them pretends.

## When to use
- Generating the templates for an ESP (Klaviyo, Mailchimp, Ecomail) with merge tags.
- Per-cart drafts when a store wants human review before sending.

## Input
Cart items (name, image, price, variant), the store's voice guide, the actual discount policy (what exists, for whom), shipping thresholds, and the support address.

## The sequence
1. **1 hour — Remind.** Subject names the product. Body: the item, one sentence on why it is a good pick (from its description), a link back to the cart. No discount.
2. **24 hours — Help.** Assume a real reason: sizing, shipping cost, a question. Answer the top two objections for this product type (from the FAQ if provided) and offer a reply-to-a-human line.
3. **72 hours — Decide.** If the policy allows a discount, state it plainly with its real end date. If not, this mail is the last reminder and says the cart will be cleared on <date>.

## Rules
- Discounts and deadlines come only from the policy in the input. Never invent urgency, a countdown, or a "limited stock" claim without a stock number.
- Never pressure: no guilt, no "we noticed you left", no fake personal sender.
- Each mail under 120 words of body. Subject under 50 characters, no emoji unless the voice guide allows.
- Merge tags in `{{ }}`; list every tag used at the end so the ESP mapping is checkable.
- One CTA per mail.

## Output format
```
### Mail 1 · +1 h
Subject: Your Speedcross 6 is still in the cart
Body: …
CTA: Back to cart → {{ cart_url }}

### Mail 2 · +24 h
…

Tags used: {{ first_name }}, {{ cart_url }}, {{ item_name }}, {{ item_image }}
```

## License
MIT
