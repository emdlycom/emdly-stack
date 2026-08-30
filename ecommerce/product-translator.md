---
name: product-translator
owner: launifycorp
category: Ecommerce
description: Localises e-shop product records into a new market: titles, descriptions, attributes, categories and slugs. Every field leaves with one of four verdicts — translated, mapped, kept, formatted — so identifiers survive, categories map to numeric IDs instead of being translated, and no size or unit is silently converted.
version: v1
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/product-translator
raw: https://emdly.com/raw/launifycorp/product-translator.md
install: npx @emdly/cli add launifycorp/product-translator
---

# Product translator

Translating a product listing is four different jobs wearing one name. Marketing copy gets translated. Attribute keys and category labels get **mapped** to the target taxonomy, never translated. Identifiers, model numbers and standards designations get **kept** byte for byte. Numbers get **formatted** for the locale, and only converted when someone decides the customer receives something different. Do all four with one pass of a translator and you ship a catalogue where the size chart is wrong and the EAN has a space in it.

Every field this skill touches leaves with one of four verdicts — `translated`, `mapped`, `kept`, `formatted` — and a reason. A field with no verdict has not been handled.

## When to use

- Opening an existing e-shop in a new market: titles, descriptions, parameters, categories, slugs.
- Adding a locale to a feed that already goes to Google Shopping, Heureka, Idealo or a marketplace.
- Re-translating a category after the source catalogue changed and the locales drifted apart.
- Auditing an existing localisation for the failure modes below, without retranslating.

Not for: translating a single marketing page (that is copy, not a listing), or for machine-translating user reviews, which this skill refuses.

## Input

Required:

- **Source records.** Per product: title, description, attribute key/value pairs, current category, current slug, and any identifiers (GTIN/EAN, MPN, SKU, brand).
- **Source and target locale**, as BCP 47 (`cs-CZ` → `de-DE`). Language alone is not enough; `de-DE` and `de-AT` differ on units, size charts and VAT wording.
- **Target category taxonomy.** For Google Shopping this is the numeric `google_product_category` ID. For a marketplace it is that marketplace's own tree.

Strongly recommended, and named in the output when absent:

- **Glossary / termbase** — how this store already translates its recurring attribute values.
- **Do-not-translate list** — brand names, product lines, trademarked feature names.
- **Existing target-locale records** for the same category, as translation memory. Consistency with what is already live beats a better isolated translation.
- **Target-market search terms**, if the store has them.

## The four verdicts

Assign one to every field before writing anything.

**`kept`** — reproduced exactly, including case and spacing. Identifiers (GTIN, EAN, MPN, SKU, ISBN), brand and product-line names, model numbers, standards designations (`IP67`, `EN 388:2016`, `USB-C`, `A2DP`), chemical and material codes, and anything on the do-not-translate list. A brand that is also a common word is still `kept`: Apple stays Apple, Orange stays Orange, Kingfisher stays Kingfisher.

**`mapped`** — replaced with the target taxonomy's own term, looked up, not translated. Category labels, attribute keys, and any attribute value drawn from a closed enum. Google's own documentation is explicit that the taxonomy is published per language and that the numeric ID is the fallback: *"If the taxonomy of product codes isn't available in your language, use the English values or numeric ID."* Submit the **numeric ID**. It is language-independent, so it cannot drift as locales are added.

**`translated`** — actual prose. Title, description, free-text attribute values, care instructions.

**`formatted`** — the number is unchanged, its presentation is not. Decimal comma, thousands separator, date order, currency position and symbol, unit spacing. Formatting never changes what the customer receives.

A **conversion** is not a verdict. Converting 10.5 UK to EU 44.5, or inches to centimetres, changes a claim about the physical product. Conversions are proposed, never applied — see Stop and hand back.

## Rules

**Grammar is not a find-and-replace.** In Czech, Polish, German and every other language with case or gender agreement, an attribute value cannot be slotted into a template and stay correct. `{barva} {produkt}` gives "červená mikina" but "červený svetr". If the store composes titles from attributes, say so in the output and hand back the template, not just the words. Do not silently produce an agreement error.

**Look up the plural rules, do not assume them.** CLDR defines six possible categories — `zero`, `one`, `two`, `few`, `many`, `other` — and which of them a language uses is a property of that language. Polish uses four. Arabic uses all six. English uses two. Any string with a count in it needs the target locale's set, from the CLDR chart, not the source locale's.

**Translated is not the same as searched.** The literal translation and the term the target market types into a search box are often different words. Where they differ, use the market's term in the title and note the literal translation in the decision log. Never stack both to cover the gap — that is keyword stuffing, and it is what gets a feed disapproved.

**Length is not preserved.** German and French running text typically runs longer than Czech or English for the same content, and the visible part of a title is short. Google's title attribute accepts 1–150 characters and its documentation states that *"users will usually notice only the first 70 or fewer characters of your title, depending on screen size."* Count the characters of the translated title, report both numbers, and put the identifying words inside the first 70. Descriptions accept 1–5,000 characters, with Google advising the important details go in the first 160–500.

**One term, one translation, everywhere.** An attribute value that appears on 400 products translates the same way on all 400. Take the term from the glossary if there is one, from existing live records if there is not, and add anything new to a glossary delta in the output.

**Slugs are derived from the translated title, then transliterated.** Lowercase, ASCII only, hyphen-separated, diacritics folded (`š` → `s`, `ř` → `r`, `ü` → `ue` for German, `ß` → `ss`). Drop articles and filler. Keep the model number if the source slug had it. Propose, never replace: see Stop and hand back.

**A slug is a URL, and URLs are a separate decision from language.** Google's guidance is that each language version must be a distinct URL and that *"each language version must list itself as well as all other language versions"* in its hreflang annotations, with `x-default` for visitors matching nothing. Google also states plainly that it *"doesn't use hreflang or the HTML lang attribute to detect the language of a page."* So a localised slug is for the reader, not for the crawler's language detection, and it is worth nothing if the hreflang set is not maintained alongside it.

**Never translate a review.** User-generated content stays in the language it was written in, or is labelled as machine-translated by the platform. This skill does not touch it.

**Report what you did not do.** Fields you could not resolve are listed with the reason. Silence is not a pass.

> Thresholds above are defaults; report the thresholds you used.

## Output format

Per product, then a run-level summary.

```
PRODUCT 4471-A · cs-CZ → de-DE

title        [translated] Herren T-Shirt Merinowolle 150 g/m² Kurzarm
                          Rundhalsausschnitt — Anthrazit
             74 chars · source was 74 · FLAG: cut at 70 gives "…— Anthr"
             colour falls outside the visible window, reorder before publish
description  [translated] 1 842 chars · key details in first 214
slug         [proposed]   merino-t-shirt-herren-150-anthrazit
             from title · folded ü→ue · PROPOSED ONLY, see Stop 3
category     [mapped]     212 (Apparel & Accessories > Clothing > Shirts & Tops)
             submitted as numeric ID, not as a label

ATTRIBUTES
key (source)      key (target)      value            verdict      note
Materiál          Material          Merinowolle      translated   glossary: Merino → Merinowolle
Gramáž            Flächengewicht    150 g/m²         formatted    non-breaking space before g/m²
Střih             Passform          Regular Fit      kept         do-not-translate list
Velikost          Größe             M                kept         letter size, no chart supplied
Barva             Farbe             Anthrazit        translated   glossary delta: new term
Certifikace       Zertifizierung    OEKO-TEX 100     kept         standards designation
Původ             Herkunft          —                —            NOT SUPPLIED in source
Praní             Pflege            (not resolved)   —            HELD, see below

HELD
Praní — source reads "Perte na 30 °C, nesušte v sušičce." The care symbols on the
label are the legal text in DE; translating the Czech sentence would produce a
second, unverified instruction. Needs the label artwork. Owner: product data.

DECISION LOG
- Title: literal "Pánské triko" → "T-Shirt Herren". The term the market searches
  is "Herren T-Shirt"; used the market term, literal recorded here.
- Title flagged, not fixed. Reordering it is a merchandising call: dropping
  "Rundhalsausschnitt" or moving "Anthrazit" forward both work, and which one
  loses less is not a translation decision.
- Title composed from attributes upstream: template `{Barva} {Produkt}` does not
  survive German adjective inflection. Template handed back, not the words.
- Size M kept, not converted. CZ M and DE M are not guaranteed the same garment.
```

The run summary:

```
RUN cs-CZ → de-DE · 418 products · 12 categories

translated 1 254 fields · mapped 836 · kept 2 090 · formatted 418
held 34 · not supplied 61

Glossary delta: 22 new terms (attached, needs sign-off before the next run)
Categories mapped to numeric IDs: 12 of 12
Titles over 70 visible chars: 47 — listed, identifying words still inside 70
Slugs proposed: 418 · slugs replaced: 0 (this skill does not replace)
Stops raised: 3 (Stop 1 ×2, Stop 5 ×1) — see below, run is not complete
```

The refusal, when a locale cannot be produced at all:

```
NOT TRANSLATED · cs-CZ → de-DE · 418 products

Blocked: no target taxonomy supplied and no glossary.
Category mapping is not a translation task and cannot be inferred from the label
text. Without the target tree, every category verdict would be a guess that looks
like a decision.

Needed: the google_product_category IDs for the 12 source categories, or the
marketplace's own tree. The 22 candidate glossary terms are attached so the
terminology decision can be made before, not after, 418 products are written.

Nothing was written. No slug was changed.
```

## Edge cases

- **Source is already machine-translated.** Say so and stop translating it. Translating a bad translation compounds the error and hides the original. Ask for the original-language record.
- **Mixed-language source.** A Czech description with English marketing lines is normal. Identify the language of each segment; do not homogenise a deliberately English product-line name into the target language.
- **HTML in the description.** Translate the text nodes only. Preserve tag structure, attribute values, entities and self-closing tags exactly. Report any tag whose text content you could not match back.
- **Merge tags and placeholders** (`{{price}}`, `%s`, `{0}`) are `kept`, and their surrounding grammar is the translator's problem: a placeholder that was a subject in the source may need a different case in the target.
- **Enum value with no target equivalent.** Do not invent one and do not fall back to the literal translation. Mark the value `HELD` and name the closest two candidates in the target taxonomy.
- **Variants sharing a parent.** Translate the parent once. Variant-only fields (size, colour) inherit the parent's terminology. Never let two variants of one product disagree on a term.
- **Product has no description.** `NOT SUPPLIED`. Do not generate one. Writing a description is not translation, and an invented one will contain claims nobody approved.
- **Source title is keyword-stuffed.** Translate the product, not the stuffing. Report what you dropped and why.
- **Text inside images.** Flag it. You cannot translate it, and a localised listing with a source-language image is a visible defect.
- **Catalogue over ~500 products.** Batch by category, so terminology decisions are made once per category rather than once per batch. State the batch boundaries in the run summary.
- **Right-to-left target locale.** Neutral text direction is not enough. Flag any field mixing RTL text with LTR identifiers or units for a human to check the rendered result.

## Stop and hand back

Stop means nothing is written for that field and the run summary records the stop. These are decisions about what the customer receives or what the store is legally saying, and they are not translation decisions.

1. **Regulated claims.** Health, medical, nutritional, safety, cosmetic and supplement claims. Allergen statements. CE marking, toy safety, electrical certification. What is a permitted claim in one market is a prohibited claim in another, and a faithful translation of an unlawful claim is still unlawful. Route to whoever owns compliance.
2. **Legal and commercial text.** Warranty terms, return and withdrawal periods, delivery promises, guarantees. Statutory return periods differ by market; a translated sentence quietly makes a promise under the wrong law.
3. **Replacing a slug that is already published.** A live URL that changes without a redirect loses its history and its links. Propose the new slug, name the redirect as a prerequisite, and let the person who owns the site's routing decide.
4. **Price, currency and tax wording.** Whether a displayed price includes VAT is a market rule, not a phrasing. Never translate a VAT-inclusive statement into a market with a different convention.
5. **Any unit or size conversion.** Report the source value, the proposed target value and the conversion used, then hand it over. A wrong shoe size is a return, not a typo. Vendor size charts are lookups, not arithmetic.
6. **Identifiers that do not survive.** A GTIN, EAN or MPN that has been reformatted, spaced or truncated anywhere upstream. Report it and do not clean it up — a repaired identifier that is wrong is worse than one that is visibly broken.
7. **No reviewer for the target locale.** Say so in the run summary. This skill produces a draft catalogue; it does not certify one.

## License

MIT
