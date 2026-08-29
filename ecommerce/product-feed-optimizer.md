---
name: product-feed-optimizer
owner: cartlift
category: Ecommerce
description: Rewrites product titles and descriptions for Google Shopping feeds — attributes first, brand rules kept, no keyword stuffing.
version: v2
license: MIT
updated: 2026-08-18
recommended: false
security_checked: true
url: https://emdly.com/skills/cartlift/product-feed-optimizer
raw: https://emdly.com/raw/cartlift/product-feed-optimizer.md
install: npx @emdly/cli add cartlift/product-feed-optimizer
---

# Product feed optimizer

Shopping ads are won in the title. This skill rewrites feed rows so the attributes people search for come first and Google's policies are never tripped.

## When to use
- On a feed export (CSV/XML) before it goes to Merchant Center.
- On the "disapproved" and "low impression" products only, weekly.

## Input
Per product: title, description, brand, product type, GTIN, color, size, material, gender, age group, price, and the landing page's H1 if available. The store's brand rules (banned words, capitalization, claims policy).

## Title formula
Pick by vertical and fill only with attributes present in the data:
- **Apparel:** Brand · Gender · Product type · Attribute (material/fit) · Color · Size
- **Electronics:** Brand · Model · Product type · Key spec · Size/capacity · Color
- **Home & garden:** Brand · Product type · Material · Dimensions · Color
- **Consumables:** Brand · Product type · Variant/flavor · Quantity/pack size

Titles under 150 characters; the first 70 carry what matters, because that is what shows.

## Rules
- Attributes come from the data, never from the title's old wording and never invented. A missing color stays missing.
- No promotional text in titles or descriptions: "sale", "free shipping", "best" — these get disapproved.
- No ALL CAPS, no exclamation marks, no emoji, no keyword lists.
- Keep brand rules: banned words out, brand casing exactly as given.
- Descriptions: 2–4 sentences, what it is, what it is for, the one attribute that decides the purchase. Under 500 characters.
- Flag rows whose landing page H1 disagrees with the title — Merchant Center compares them.

## Output format
CSV-compatible rows with `id, title, description, flags`:
```
8841, "Salomon Men's Trail Running Shoes Speedcross 6 Gore-Tex Black 43", "Waterproof trail runner with a lugged Contagrip sole for mud and loose ground. Gore-Tex lining keeps feet dry on wet trails; the quicklace system tightens with one pull.", ""
8842, "Merino Hiking Socks Crew 3-Pack Grey L", "…", "landing H1 says 'Hiker Socks' — align"
```

## License
MIT
