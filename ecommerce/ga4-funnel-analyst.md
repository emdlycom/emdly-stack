---
name: ga4-funnel-analyst
owner: shopmetric
category: Ecommerce
description: Reads a GA4 export and finds where the checkout funnel leaks — drop-off by step, device and source — with the one fix to try first.
version: v3
license: MIT
updated: 2026-08-22
recommended: true
security_checked: true
url: https://emdly.com/skills/shopmetric/ga4-funnel-analyst
raw: https://emdly.com/raw/shopmetric/ga4-funnel-analyst.md
install: npx @emdly/cli add shopmetric/ga4-funnel-analyst
---

# GA4 funnel analyst

Funnel reports tell you *that* people leave. This skill tells you *where it is worst*, *for whom*, and *what to change first*.

## When to use
- Weekly, on a GA4 export of the ecommerce events (`view_item`, `add_to_cart`, `begin_checkout`, `add_shipping_info`, `add_payment_info`, `purchase`).
- After a checkout release, comparing the week before and after.

## Input
Event counts per step, broken down by device category and session source/medium, for the period and a comparison period. Optional: page load times per step.

## Process
1. **Sanity check the data.** Each step must be ≤ the previous one. If `purchase` > `add_payment_info`, the tagging is broken — report that and stop; a funnel on bad events is worse than no funnel.
2. **Compute step conversion** for the whole period, then per device, then per source. Report absolute counts next to percentages.
3. **Find the worst leak**: the step whose conversion is lowest *relative to the site's own baseline* (the comparison period), not relative to industry numbers.
4. **Find who leaks most** at that step. A segment counts only with ≥ 200 sessions entering the step; smaller segments are listed as "too small to judge".
5. **Pick one fix.** The smallest change that addresses the worst leak for the biggest segment. One, not a list.

## Rules
- Percentages without counts are forbidden. "Mobile checkout converts 12% worse" must carry "(1 840 → 1 620 sessions)".
- Never compare against benchmarks from memory. The only baseline is the site's own comparison period.
- Correlation is a hypothesis: "mobile drops at shipping" is a place to look, not a diagnosis. Say what would confirm it (a session recording, a form-error log).
- Do not recommend more than one change. If the second-worst leak is close, mention it as next week's candidate.

## Output format
```
## Funnel — 1–7 Sep vs 25–31 Aug
| step | sessions | conv. | Δ vs prev |
| add_to_cart → begin_checkout | 12 400 → 4 960 | 40.0% | −3.1 pt |
| begin_checkout → add_shipping_info | 4 960 → 3 210 | 64.7% | −9.8 pt |  ← worst leak
...

## Where it leaks
Mobile · organic: 58.1% (1 320 → 767) vs desktop 74.2%. Paid social mobile too small to judge (140).

## Try first
Shipping step on mobile: the address form. Confirm with form-error events on `address_line_1`; if errors > 15% of attempts, enable autocomplete and ship.
```

## License
MIT
