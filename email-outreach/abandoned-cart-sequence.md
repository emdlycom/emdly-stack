---
name: abandoned-cart-sequence
owner: cartlift
category: Email & outreach
description: Drafts a three-mail abandoned-cart sequence from the cart contents and the store's voice — useful, specific, and honest about discounts.
version: v4
license: MIT
updated: 2026-08-30
recommended: false
security_checked: false
url: https://emdly.com/skills/cartlift/abandoned-cart-sequence
raw: https://emdly.com/raw/cartlift/abandoned-cart-sequence.md
install: npx @emdly/cli add cartlift/abandoned-cart-sequence
---

# Abandoned cart sequence

Three mails, each with a different job: remind, help, decide. None of them pretends. The output is ESP-ready copy that a store can load and send without a copywriter touching it, which means every claim in it — a discount, a deadline, a stock level, a return window — has to come from the input rather than from the writing. If it is not in the input, it does not go in the mail.

## When to use
- Generating templates for an ESP (Klaviyo, Mailchimp, Ecomail) with merge tags.
- Per-cart drafts when a store wants human review before sending.
- Rewriting a sequence that leans on countdown timers and "we noticed you left".
- After a policy change that makes the current mails wrong about discounts or returns.

## Input

Required:
- **Cart items** — name, variant, price, quantity, image URL.
- **Discount policy** — what discounts exist, for whom, the code, and the real end date. "None" is a valid and complete answer.
- **Cart expiry** — when the cart clears, or "never". Mail 3's no-discount branch needs a date.
- **Returns and shipping** — window, who pays return postage, free-shipping threshold.
- **Support address** — a real reply-to.
- **Consent basis** — how this address was collected, and confirmation that the suppression and unsubscribe lists are wired into the ESP.

Optional:
- Voice guide, product FAQ, stock counts.

Without the voice guide, write plain and say so in the output header. Without the FAQ, mail 2's objections come from the product type and are marked `(not stated — from product type)`. Without the discount policy or the consent basis, the skill does not run; see Stop and hand back.

## The cadence

1 h / 24 h / 72 h.

**This has no citable source.** Every "optimal send time" figure in circulation is published by an ESP that is paid per send. Do not repeat a percentage lift for any timing, and do not present these three delays as a finding.

What they are actually anchored on:
- **1 h** — long enough that the person has left the site, short enough that the cart is still the thing they were doing. [judgment]
- **24 h** — the same hour of the next day, so it lands in a comparable moment. [judgment]
- **72 h** — inside the cart TTL, so mail 3 can name a real expiry date instead of inventing urgency. [judgment, anchored on the store's own cart expiry]

If the store has its own data, replace all three: take carts that eventually converted, read the distribution of time from abandon to purchase, put mail 1 before its first quartile and mail 3 after its median.

> Thresholds above are defaults; report the thresholds you used.

## Length limits

- **Body under 120 words**, counting from the greeting to the last sentence before the CTA. Excludes subject, CTA line, annotations and the tag list. A merge tag counts as one word. House rule.
- **Subject under 50 characters.** No standard sets a subject length. Mail clients truncate the preview by available pixel width, not by character count, and that width differs per client and device — so a character cap is a proxy, not a rule. 50 is the narrowest common mobile preview. [judgment]
- **One CTA per mail.** House rule.

## The sequence

1. **+1 h — Remind.** Subject names the product. Body: the item and variant, one sentence on why it is a good pick drawn from its own description, and a link back. No discount. No question.
2. **+24 h — Help.** Assume a real reason. Answer the top two objections for this product type — from the FAQ when supplied, from the product type when not — and offer a reply-to-a-human line pointing at the support address.
3. **+72 h — Decide.** Two branches, and the skill writes whichever the policy dictates:
   - **Discount branch.** State the code and the real end date from the policy. Nothing else changes.
   - **No-discount branch.** This is the last reminder. It says the cart clears on the supplied expiry date, and it says plainly that there is no code. Never soften this into a hint that one might arrive.

## Rules

- Discounts, deadlines, stock claims and return terms come only from the input. No countdown, no "limited stock" without a stock number, no "price going up".
- No pressure and no guilt: no "we noticed you left", no "did you forget?", no fake personal sender name, no fabricated first-person aside.
- Every mail carries the unsubscribe link and the store's postal address. The US CAN-SPAM Act (15 U.S.C. §7704, implemented at 16 CFR Part 316) requires a working opt-out mechanism and a valid physical postal address in commercial email. For EU recipients, ePrivacy Directive 2002/58/EC Art. 13(2) permits these mails to existing customers about similar products only, with an opt-out offered at collection and in every message.
- Merge tags in `{{ }}`. List every tag used at the end so the ESP mapping is checkable. A tag the input cannot fill is not invented.
- Sequence exits on purchase and on unsubscribe. State both exits in the output.
- One CTA per mail, and it goes to the cart, never to a category page.

## Output format

A header stating the delays and thresholds used, then all three mails in full, then the tag list and exits. No placeholders — if a mail cannot be written, say which input is missing instead of shipping a stub.

```
## Abandoned cart · Salomon Speedcross 6, size 43, €150 · 1 cart item
Delays used: 1 h / 24 h / 72 h (defaults — no store timing data supplied)
Limits used: body ≤ 120 words, subject ≤ 50 chars, 1 CTA
Voice guide: supplied. FAQ: not supplied — mail 2 objections from product type.
Discount policy: none for footwear. Mail 3 uses the no-discount branch.

### Mail 1 · +1 h · Remind
Subject: Your Speedcross 6 is still in the cart          [38 chars]
Body:
Hi {{ first_name }},

You left the Salomon Speedcross 6 in size 43 in your cart.

It is the shoe people buy for mud and loose ground. The deep lugs bite where a
road-ish trail shoe slides, and the Gore-Tex lining means wet grass does not end
the run early.

Your cart is where you left it.
                                                        [55 words]
CTA: Back to cart → {{ cart_url }}

### Mail 2 · +24 h · Help
Subject: Sizing and returns on the Speedcross 6          [38 chars]
Body:
Hi {{ first_name }},

Two things people ask before buying these.

Sizing. They run about half a size small and narrow through the midfoot. Between
sizes, or a wide foot, take the next size up.

Returns. Thirty days, unworn, and we pay the return label. Trying them on indoors
does not count as worn.

If your question is neither of those, reply to this mail. It goes to
{{ support_email }} and a person answers it.
                                                        [72 words]
[objections: not stated — from product type]
CTA: Back to cart → {{ cart_url }}

### Mail 3 · +72 h · Decide — NO-DISCOUNT BRANCH (policy: no footwear discounts)
Subject: Last note about your Speedcross 6               [33 chars]
Body:
Hi {{ first_name }},

Last mail about this one.

There is no code to offer. We price the Speedcross 6 the same for everyone, all
year, so what you saw is what it costs.

Your cart holds until {{ cart_expiry_date }}. After that it clears and the size 43
goes back to general stock.

If you have decided against it, no reply needed. Unsubscribe below and these stop.
                                                        [64 words]
CTA: Back to cart → {{ cart_url }}

### Mail 3 · alternate — DISCOUNT BRANCH (written only if the policy supplies a code)
Subject: 10% on your cart until Friday                   [29 chars]
Body:
Hi {{ first_name }},

Last mail about this one.

The Speedcross 6 in size 43 is still in your cart. The code
{{ discount_code }} takes 10% off it.

It stops working on {{ discount_end_date }}. That is the real date from the
policy, not a rolling timer, and it will not be extended.

If the shoe is not right, that is a fine answer. Unsubscribe below and these stop.
                                                        [64 words]
CTA: Back to cart → {{ cart_url }}

Tags used: {{ first_name }}, {{ cart_url }}, {{ support_email }},
{{ cart_expiry_date }}, {{ discount_code }}, {{ discount_end_date }}
Exits: purchase · unsubscribe
Footer required on all three: unsubscribe link + store postal address
```

The decline, when the discount policy is absent, is the whole output:

```
## Declined — no discount policy supplied
Mail 3 branches on the policy and there is no safe default. Guessing "no discount"
writes a false statement into a customer-facing send; guessing a discount invents
money. Mails 1 and 2 are not shipped alone — a two-mail sequence with no close is a
different sequence, not a partial one.
Supply: what discounts exist, for whom, the code, the end date. "None" is a complete
answer and produces the no-discount branch above.
Mails written: 0.
```

## Edge cases

- **Empty cart.** No item, no mail. Report `cart empty — nothing to remind about` and stop. Do not substitute a browse-abandonment mail; that is a different sequence with a different consent basis.
- **Oversized cart** — more than 5 line items. Naming every item blows the 120-word limit. Name the highest-priced item and the count: "your Speedcross 6 and 6 other items". Do not raise the word limit to fit the list. [judgment]
- **No discount policy supplied.** Decline as above.
- **Policy supplies a discount with no end date.** Do not write "for a limited time". Ask for the date, or write the no-discount branch and say why.
- **No cart expiry supplied.** The no-discount branch has nothing true to say about timing. Drop the expiry sentence rather than inventing a date, and mark the mail `no expiry stated — closing line removed`.
- **No voice guide.** Write plain, no emoji, no exclamation marks, and state `voice guide: not supplied` in the header. Do not imitate a voice from the product copy.
- **No FAQ.** Mail 2 still runs, objections from the product type, each marked `(not stated — from product type)` so a human can check them.
- **No support address.** Cut the reply-to-a-human line from mail 2 rather than pointing at an address that does not exist. Mail 2 is then two objections and a CTA.
- **Cart contains a mix of regulated and ordinary items.** The whole cart is held. See below; you cannot mail about half a cart.

## Stop and hand back

Halt and name who decides. These are customer-facing sends that go out without further review.

1. **A discount that could put an item below cost.** If the policy's percentage applied to any cart item crosses the cost figure supplied, or if no cost figure was supplied at all, do not write the discount branch. Hand back with the item, its price, and the discount asked for. The store's finance owner decides.
2. **Regulated or age-restricted items in the cart** — alcohol, tobacco and vaping, supplements, medical devices, drugs, weapons, adult products. Marketing wording in these categories carries legal exposure and some jurisdictions restrict the send outright. Hold the whole sequence and hand to whoever owns compliance.
3. **Consent, suppression or unsubscribe not established.** If the input does not say how the address was collected, or does not confirm the suppression and unsubscribe lists are wired into the ESP, do not produce sendable copy. This is the CAN-SPAM and ePrivacy obligation above, and it is not a copy decision. Hand back to whoever owns the ESP.
4. **A cart that already converted, or an address on a suppression list.** Do not write the sequence. Report it as a targeting error in the trigger, not a copy request.
5. **The store asks for urgency the input does not support** — a countdown, a stock warning, a "price rises tomorrow". Refuse the line, say which input would make it true, and hand the decision back.

## License
MIT
