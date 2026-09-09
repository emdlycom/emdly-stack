---
name: product-translation-audit
owner: launifycorp
category: Ecommerce
description: Scores the quality of a product catalogue's translations market by market, and returns concrete rewrites for what drags the score down. It judges a translation the way a shop owner should: not "is it...
version: v1
license: MIT
updated: 2026-09-09
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/product-translation-audit
raw: https://emdly.com/raw/launifycorp/product-translation-audit.md
install: npx @emdly/cli add launifycorp/product-translation-audit
---

# Product Translation Audit

Scores the quality of a product catalogue's translations market by market, and returns concrete rewrites for what drags the score down. It judges a translation the way a shop owner should: not "is it correct?" but "does it sell, does it get found, and will it get us in trouble?"

A literal, grammatically perfect translation can still be a bad product page. This skill is built to catch exactly that.

## When to use

- "how good are our German product translations", "audit the product copy for the French shop"
- After a machine-translation run, a new market launch, or a PIM migration.
- When a locale converts or ranks worse than the source market and nobody knows why.

Do not use it to translate a catalogue — this audits and recommends. Bulk translation is a separate job.

## Step 1 — Establish the inputs

- **Source of data**: a PIM or shop export (CSV/XLSX/JSON), a connected store API, or pasted product records. Ask for the export if nothing is connected.
- **Fields in scope**: product title, short and long description, bullet points, attributes and their values, variant names, category names, meta title and description, URL slug, alt text.
- **Source locale** and **target locales** — and which market each serves. `de-DE`, `de-AT` and `de-CH` are different markets with different terms, sizes, and price expectations; never audit them as one.
- **Sampling.** Catalogues are large and quality is not uniform. Audit a stratified sample, not the first N rows: the top sellers by revenue, a slice of the mid tail, a slice of the long tail, plus every product in any category launched recently. State the sample size and how it was chosen. Never present a sample's score as the whole catalogue's without saying so.

## Step 2 — Score each locale on seven dimensions

Score 1–5 per dimension with a one-line justification and real examples. Weight them by commercial impact, not by how easy they are to measure:

| # | Dimension | Weight | What fails it |
|---|---|---|---|
| 1 | **Accuracy** | 20% | Wrong meaning, dropped qualifiers, invented features, mixed languages in one field |
| 2 | **Search fit** | 25% | Translated the word instead of the term buyers actually search for in that market |
| 3 | **Persuasiveness** | 20% | Reads like a translation: literal source syntax, no benefit framing, dead CTA |
| 4 | **Consistency** | 15% | The same attribute or product term rendered differently across the catalogue |
| 5 | **Formats and units** | 10% | Sizes, measurements, currency, decimal separators, dates left in source convention |
| 6 | **Technical integrity** | 5% | Broken placeholders, stray HTML, truncated fields, encoding damage |
| 7 | **Compliance surface** | 5% | Legally required information missing or left in the source language |

Report the weighted total per locale, plus the per-dimension breakdown. A single number hides where the money is.

## Step 3 — What to look for, in detail

**Search fit — usually the biggest lever.** Buyers search their own vocabulary, not a dictionary's. German shoppers type *Turnschuhe* and *Sneaker* in different contexts; British shoppers say *trainers*, not *sneakers*; a *jumper* is not a *sweater*. Flag every title and H1 where the head term is a literal translation of the source rather than the market's term. If a keyword tool or Search Console is connected, use its data. If it is not, say so and label the suggestion a hypothesis to validate — never state search volumes from memory.

**Consistency.** Attribute values drive faceted navigation. If *Navy* is *Marineblau* on one product and *Dunkelblau* on another, the colour filter silently splits the catalogue in two. Build a term frequency table per attribute and flag every value with competing variants. The same applies to recurring marketing phrases and category names.

**Untouchables.** Brand names, model designations, SKUs, and certification marks stay exactly as they are — *Air Max* is not *Luft Max*. Flag anything translated that should not have been. Material composition and care symbols are usually regulated wording, not copy.

**Length budgets.** German runs roughly 20–35% longer than English, Finnish longer still; CJK much shorter. Check every field against its real limit — the PDP layout, and any channel the feed goes to (marketplace title caps, Google Shopping title and description limits). A title that truncates mid-word costs clicks.

**Formats.** Sizes need market conversion (US 10 / UK 9 / EU 43), not translation. Check measurement units, currency and its position, decimal and thousands separators, date order, and address or phone formats in shipping copy.

**Machine-translation tells.** Source word order preserved intact; formal/informal register flipping within one page; a glossary term translated three ways in three paragraphs; idioms rendered literally; gender agreement wrong on adjectives. List them as examples, not as a verdict on the whole locale.

**Compliance surface.** Note where legally required information appears absent or untranslated — ingredients, allergens, safety warnings, energy labels, warranty and return terms. Report it as *needs legal review in that market*, with the field and the product. This skill flags gaps; it does not give legal advice and never certifies a listing as compliant.

## Step 4 — Recommend, concretely

Findings without a rewrite are noise. For each issue give:

- the product and field,
- the current text,
- a proposed rewrite,
- one line on why it is better, in commercial terms,
- effort (single field / template / catalogue-wide) and expected impact (high / medium / low).

Group by **fix pattern**, not by product: "the head term *Sneaker* should be *Turnschuhe* in 340 titles" is one decision, not 340. Lead with the patterns that touch the most revenue.

Rank the work: what to fix this week (high impact, mechanical), what needs a native reviewer, and what is cosmetic. Say plainly which locales are healthy enough to leave alone.

## Rules

- Never invent search volumes, competitor data, or market statistics. Either it comes from connected data, or it is labelled a hypothesis.
- Never rate a translation on grammar alone — a flawless sentence that misses the market term still fails.
- Never claim a listing is legally compliant. Flag gaps for a human with jurisdiction knowledge.
- Never modify the catalogue. This skill produces a report; applying changes is a separate, confirmed action.
- Never assume one locale per language. `pt-PT` and `pt-BR`, `es-ES` and `es-MX`, `zh-Hans` and `zh-Hant` are separate audits.
- Where the source copy itself is weak, say so — translating bad copy faithfully produces bad copy in every market.
- Keep every quoted example verbatim, including the errors. Do not silently clean up what you quote.

## Output format

```
Catalogue: <name>   Source: <en-GB>   Sampled: <n> of <N> products (<method>)

SCORECARD
  locale   total   acc  search  persu  consis  format  tech  compl
  de-DE     3.4     4      2       3      3       4      5     3
  fr-FR     4.1     4      4       4      4       4      5     4

TOP PATTERNS TO FIX
  1  de-DE  Search fit   340 titles use "Sneaker" where "Turnschuhe" is the
            category term buyers search — hypothesis, validate with Search Console
            Effort: template   Impact: high
  2  de-DE  Consistency  Colour "Navy" appears as Marineblau / Dunkelblau /
            Navy across 87 products — splits the colour filter
            Effort: catalogue-wide find-replace   Impact: high

EXAMPLE REWRITES
  SKU 88213 · de-DE · title
    now: "Herren Sneaker Leder Schwarz Komfortabel Atmungsaktiv"
    to:  "Herren Turnschuhe Leder Schwarz - atmungsaktiv, 42-46"
    why: market head term, keyword stuffing removed, size range added (filter + trust)

NEEDS LEGAL REVIEW (<n>)
  SKU 41902 · de-DE · care instructions left in English

HEALTHY: fr-FR, nl-NL — no action recommended
```

## License

MIT
