---
name: product-feed-optimizer
owner: cartlift
category: Ecommerce
description: Rewrites product titles and descriptions for Google Shopping feeds — attributes first, brand rules kept, no keyword stuffing.
version: v5
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/cartlift/product-feed-optimizer
raw: https://emdly.com/raw/cartlift/product-feed-optimizer.md
install: npx @emdly/cli add cartlift/product-feed-optimizer
---

# Product feed optimizer

Shopping ads are won in the title. This skill rewrites feed rows so the attributes people search for come first, so nothing in the row is invented, and so the row survives Merchant Center's editorial and policy checks. The output goes into a live feed at scale, which means every value must be traceable to a field that was supplied. A row the skill cannot source is left alone and flagged, not improved.

## When to use
- On a feed export (CSV/XML) before it goes to Merchant Center.
- Weekly, on the "disapproved" and "low impression" slices only.
- After a catalog import that overwrote titles with supplier strings.
- When a vertical's titles were written by hand and drift from the formula.

## Input

Required, per product:
- `id`, `title`, `description`, `brand`, `product type`, `price`

Required, once for the run:
- The store's brand rules: banned words, exact brand casing, claims policy.

Optional, used when present, never guessed when absent:
- `gtin`, `color`, `size`, `material`, `gender`, `age group`, `pattern`, `capacity`, the landing page `H1`, the landing page price and availability.

If the brand rules are missing, the method collapses: casing and banned words are the two rules the skill cannot derive from the row. Decline the run and say so (see Edge cases). If a per-row optional attribute is missing, the slot is dropped from the formula — it is never filled from the old title's wording, from the description, or from inference.

## Character limits

These are Google's, not house numbers.

- Title: **1–150 characters.** Google Merchant Center product data specification, `title [title]`.
- **First ~70 characters carry the row.** Google: "Users will usually notice only the first 70 or fewer characters of your title, depending on screen size." Same page. Everything that decides a click goes before character 70.
- Description: **1–5,000 characters** is the spec maximum. Google Merchant Center product data specification, `description [description]`.
- **House rule: cap descriptions at 500 characters.** The spec allows ten times that. Google's own guidance is to put the important details in "the first 160–500 characters" because the rest is behind a click, so writing past 500 buys nothing. House rule, anchored on that guidance.
- House rule: descriptions run 2–4 sentences. Nothing in Google's spec sets a sentence count.

Sources:
- https://support.google.com/merchants/answer/6324415 (title)
- https://support.google.com/merchants/answer/6324468 (description)

> Thresholds above are defaults; report the thresholds you used.

## Title formula

Pick by vertical. Fill a slot only from a supplied attribute; drop the slot otherwise. Never reorder to fill a gap.

- **Apparel & footwear:** Brand · Gender · [Model] · Product type · Attribute (material/fit/tech) · Color · Size
- **Electronics:** Brand · Model · Product type · Key spec · Capacity/size · Color
- **Home & garden:** Brand · Product type · Material · Dimensions · Color
- **Consumables:** Brand · Product type · Variant/flavor · Quantity/pack size

If the assembled title exceeds 150 characters, drop slots from the right until it fits. Never truncate mid-word, and never drop Brand or Product type — a row that cannot fit both is flagged `too long — needs a shorter product type`, not shipped.

## Rules

- Attributes come from the supplied data. A missing color stays missing. Do not read a size out of the old title.
- No promotional text anywhere in title or description: "sale", "free shipping", "best", "%", "deal", price strings. Google's description guidance names price information, sale dates and shipping details as things to leave out; promotional text in titles is a routine disapproval.
- No ALL CAPS (a word of 4+ letters entirely uppercase, brand acronyms excepted), no exclamation marks, no emoji, no repeated keyword lists.
- Brand casing exactly as the brand rules give it. Banned words removed, and the removal noted in `flags`.
- Descriptions say what it is, what it is for, and the one attribute that decides the purchase. No links, no competitor comparisons, no company name.
- **Price and availability that disagree with the landing page are a documented disapproval** — Merchant Center crawls the page and raises `Mismatched value (page crawl)` on `price` and `availability`. Flag these and do not touch the row.
- **A title/H1 disagreement is not a documented Merchant Center check.** It is a relevance signal worth flagging, but do not claim Google compares them. Flag as `H1 mismatch (house rule)`.

## Batching

Feeds run to tens of thousands of rows. Do not attempt a whole feed in one pass.

- Process in batches of **500 rows**. [judgment, anchored on keeping a single reviewable diff]
- Above **5,000 rows in scope**, stop and ask which slice to run: disapproved, low-impression, or one product type. Do not sample silently — a sampled feed export that gets uploaded is a partial feed.
- If asked for a whole feed anyway, produce batch 1 and the count of remaining batches, then wait.

## Output format

CSV-compatible rows, `id, title, description, flags`, followed by a run summary. `flags` is empty for a clean rewrite. Every rewritten title is followed by its character count and every description by its own.

```
## Batch 1 of 1 — 5 rows in scope
Thresholds used: title ≤ 150 (spec), description ≤ 500 (house), batch 500.

8841, "Salomon Men's Speedcross 6 Trail Running Shoes Gore-Tex Black 43", "Waterproof trail runner with a lugged Contagrip sole for mud and loose ground. The Gore-Tex lining keeps feet dry on wet trails, and the quicklace system tightens with one pull.", ""
   title 64 chars · description 177 chars

8842, "Merino Hiking Socks Crew 3-Pack Grey L", "Merino-blend crew socks for multi-day walking. The wool moves moisture off the skin so they stay wearable on a second day, and the cushioned heel and toe take the rub from stiff boots.", "brand missing — title starts at product type; H1 mismatch (house rule): page H1 says 'Hiker Socks'"
   title 38 chars · description 184 chars

9107, (held), (held), "regulated category (supplement) — not rewritten, see Stop and hand back"

9250, (held), (held), "feed price 89.00 vs landing page 79.00 — Mismatched value (page crawl) [price]; fix the source, then re-run"

9333, "Anker Prime 250W 6-Port GaN Desktop Charger Black", "Six-port desktop charger that runs a laptop and five smaller devices at once from a single wall socket. GaN internals keep it about the size of a paperback. Two USB-C ports hold 140 W each.", ""
   title 49 chars · description 189 chars

## Run summary
rewritten: 3 · held: 2 · banned words removed: 0 · titles over 150 before rewrite: 0
```

The decline, when brand rules are absent, is the whole output:

```
## Declined — no brand rules supplied
This skill cannot run without banned words, brand casing and the claims policy.
Casing and banned words are not derivable from a feed row, and guessing them writes
policy violations into a live feed at scale.
Supply the three, or name a single product type and I will flag rows only.
Rows in scope: 4 812. Rewritten: 0.
```

## Edge cases

- **No brand rules supplied.** Decline as above. Do not fall back to "sensible defaults" — title case on a brand that ships lowercase is a brand-rule violation the store will not catch at feed scale.
- **Feed empty or zero rows in scope.** Report `rows in scope: 0` and stop. Do not widen the filter to find work.
- **Malformed export** — unescaped commas, broken XML, a header row that does not match the fields. Report the first 3 offending row numbers and stop. Do not repair a feed by guessing column boundaries.
- **Over 5,000 rows.** See Batching. Ask for a slice.
- **No landing page data.** Skip the price/availability check and say so in the summary: `page crawl check: not run, no landing page data`. Do not report the rows as clean.
- **No `product type`.** The formula has no anchor. Flag `product type missing — cannot apply formula` and leave the row untouched.
- **Title already compliant.** Leave it. Report it as `unchanged` in the summary; do not rewrite for the sake of a diff.
- **Description already over 5,000 characters.** That is a spec violation, not a style question. Flag it and hand back the row rather than cutting 4,500 characters of someone's copy on your own.
- **Non-Latin or mixed-language rows.** The 70-character rule is a display heuristic Google states for its own rendering; do not apply the character formula to a script where it does not hold. Flag `language out of scope`.

## Stop and hand back

Halt the row (or the run) and name who decides. Do not rewrite and flag — hold the row out of the output.

1. **Regulated or age-restricted category** — supplements, medical devices, drugs, alcohol, tobacco and vaping, weapons, adult products, financial products. Title and description wording in these categories carries legal and policy exposure that is not a feed question. Hold the rows, list their ids, and hand to whoever owns the store's compliance.
2. **Feed data and landing page disagree on a factual attribute** — price, availability, GTIN, capacity, count. Hold the row. The fix is in the source system, not the title. Name which field and both values.
3. **Before any bulk run.** A batch over 500 rows, or any run that will be uploaded without a human reading the diff, needs an explicit go from the person who owns the Merchant Center account. Produce the batch, state the row count, and wait.
4. **A brand rule and a Google policy conflict** — a mandated claim ("clinically proven", "#1") that reads as promotional or unsubstantiated. Do not pick a side. Name both rules and hand to the store owner.
5. **A rewrite that changes what the product is** — a different model number, a different capacity, a different pack size than the old title claimed. One of the two is wrong about the catalog. Hold and ask.

## License
MIT
