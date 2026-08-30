---
name: search-console-auditor
owner: shopmetric
category: Ecommerce
description: Audits Google Search Console data — pages losing clicks, queries with impressions but no CTR, and product pages cannibalizing each other.
version: v5
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/shopmetric/search-console-auditor
raw: https://emdly.com/raw/shopmetric/search-console-auditor.md
install: npx @emdly/cli add shopmetric/search-console-auditor
---

# Search Console auditor

Search Console has the answers; it just does not sort them. This skill produces the three lists that are worth acting on and ignores the rest. The reader is whoever will spend a week fixing pages, so a list is only useful if every row on it clears a stated threshold and carries the counts that prove it. Three empty lists is a legitimate and complete result.

## When to use
- Monthly, on a 28-day export against the previous 28 days.
- After a migration, a template change or a title rewrite.
- When organic traffic moved and nobody knows whether demand, rank or CTR moved it.
- Before a content sprint, to establish which existing pages are bleeding.

## Input

Required — a Search Console performance export, one row per **query × page**, for two periods of equal length, with:
- `query`, `page`, `clicks`, `impressions`, `ctr`, `position`, for the current period and the comparison period.

Required alongside it:
- The list of brand terms, so lists 2 and 3 can exclude them.

Optional:
- Weekly (not just period-total) rows, needed for list 3's ranking-swap test.
- The current `<title>` of each page, needed to quote it in list 2.
- Any migration, template change or index change inside either period.

Without weekly rows, list 3 cannot run — the swap test is the whole method and impression share alone does not establish cannibalization. Report list 3 as `not run — weekly rows required` rather than substituting a weaker test. Without the brand list, run lists 2 and 3 anyway but mark them `brand terms not excluded` at the top.

## The three lists

**1. Losing pages.** Pages whose clicks fell **≥ 20%** *and* **≥ 50 clicks in absolute terms**. Both conditions, not either. For each: did impressions fall (demand or indexing) or did CTR and position fall (the page or the SERP changed)? Say which.

**2. Free clicks.** Queries with **≥ 1 000 impressions**, **position ≤ 8**, and CTR **under half this site's own average for that position band**. These are titles and descriptions that do not match the intent. Quote the current title.

**3. Cannibalization.** A query where two URLs each take **≥ 10% of its impressions in both periods**, *and* the ranking URL swaps between weeks. Name both URLs and which should win: the higher-CTR one, unless it is a category page fighting a product page, in which case the category wins.

## Where the thresholds come from

> Thresholds above are defaults; report the thresholds you used.

- **20% and 50 clicks together.** Clicks over a fixed window behave roughly as counts, so the noise band scales with the square root. The pair only admits a page whose baseline is at least 250 clicks (20% of 250 = 50). At 250 clicks the 95% Poisson band is ±31 clicks; a 50-click fall is 1.6× that. Either condition alone lets noise in — 20% of 40 clicks is 8 clicks, which is nothing. [judgment, anchored on the Poisson band at n=250]
- **1 000 impressions.** At 1 000 impressions and a 4.8% band CTR, the 95% band on the observed CTR is about ±1.3 points. "Under half the band average" is a 2.4-point gap, comfortably outside it. Below 1 000 impressions that stops being true. [judgment, anchored on the binomial band at n=1 000]
- **Position ≤ 8.** Above position 8 the lever is rank, not the title. This is a house rule about which fix the list is for, not a claim about click curves. House rule.
- **10% impression share.** A filter to keep near-zero URLs out of list 3. The swap between weeks is the actual signal. House rule.
- **Position band CTR averages come from this site's own export.** Bands: 1 · 2 · 3 · 4–5 · 6–8. **Do not use a published CTR-by-position curve.** Those studies disagree by several points at every position and none of them controls for the SERP features on your queries. If the site has fewer than 1 000 impressions in a band, that band has no usable average — say so and skip its queries.
- **Position changes.** Average position is impression-weighted, so it moves when the query, device or country mix moves even with no ranking change. Never call a change under 0.3 a drop (house rule), and never call any position change a drop while impressions moved more than 20% — report both numbers and say the mix moved.

## Rules

- Every figure carries its counts. "Clicks fell 35%" alone is not a row; "1 240 → 810 (−34.7%, −430)" is.
- Brand queries are excluded from lists 2 and 3 — they behave differently. If a brand query moved, list it in a separate `Brand` block and do not rank it against the others.
- Do not suggest content to write. This skill finds problems on existing pages. A missing page is out of scope; say so if asked.
- Do not attribute a movement to an algorithm update. The export cannot show that. Name what changed on the page or in the SERP, or say `cause not established in this data`.
- Diagnose each losing page on one side only: impressions moved, or CTR and position moved. If both moved, say both moved and do not pick.
- A migration, template change or index change inside either period is stated at the top and beside every affected row.
- Every list ends with its count, including zero.

## Output format

A header stating periods and thresholds, then the three lists, then the brand block. Each list header carries its count.

```
## Search Console — 1–28 Sep vs 4–31 Aug
Thresholds used: losing ≥ 20% and ≥ 50 clicks · free clicks ≥ 1 000 impr, pos ≤ 8,
CTR < half band avg · cannibalization ≥ 10% impr share both periods + weekly swap.
Band CTR averages from this export. Brand list supplied. No migrations in either period.

## Losing pages (2)
- /shoes/trail-runners — clicks 1 240 → 810 (−34.7%, −430). Impressions 41 300 → 40 900
  (−1.0%, flat). Position 4.1 → 7.8 (+3.7). Rank side: the page lost position, demand did
  not move. Page unchanged since March; 2 new competitors now in the top 5.
- /gear/headlamps — clicks 620 → 430 (−30.6%, −190). Impressions 18 400 → 12 100
  (−34.2%, −6 300). Position 3.8 → 3.9 (+0.1, under 0.3 — not a drop). Demand side:
  fewer impressions at the same rank. Seasonal or an index change; cause not established
  in this data.

## Free clicks (2)
- "waterproof trail running shoes" — 14 200 impr., 156 clicks, CTR 1.1%. Position 5.2
  (band 4–5, site average 4.8%; half is 2.4%). Title: "Trail Runners | Shop" — says
  nothing about waterproof.
- "trail shoes for mud" — 3 100 impr., 28 clicks, CTR 0.9%. Position 6.9 (band 6–8, site
  average 2.6%; half is 1.3%). Title: (not stated) — no title supplied for
  /shoes/trail-runners in this export.

## Cannibalization (1)
- "merino socks" — 9 400 impr. total. /socks/merino (category) 4 100 (43.6%),
  /socks/merino-hiker-2 (product) 2 800 (29.8%). Both clear 10% in both periods, and the
  ranking URL swapped 3× in 4 weeks. Winner: the category. Add the product to its top
  grid and link back from the product.

## Brand (1, not ranked)
- "cartlift trail runners" — 8 900 → 9 400 impr., clicks 2 100 → 2 240. Moved up; no
  action.
```

The empty result is a complete output, not a failure:

```
## Search Console — 1–28 Sep vs 4–31 Aug · small site
Thresholds used: defaults, as above.
14 pages received any clicks in either period.

## Losing pages (0)
Nothing clears both conditions. The largest fall is /guides/sizing, 62 → 55 clicks
(−11.3%, −7). It fails on both: under 20% and under 50 clicks.

## Free clicks (0)
No query reaches 1 000 impressions. The largest is "merino sock sizing" at 340.
Band 4–5 holds 610 impressions site-wide, under 1 000, so it has no usable CTR average
either — the test cannot be run on this site at these thresholds.

## Cannibalization (0)
No query has two URLs at ≥ 10% impression share in both periods.

## What this means
The site is below the size these thresholds are built for. Nothing here is a finding,
and nothing here says the site is healthy — it says the data is too thin to rank.
Two options, and neither is done silently:
- Re-run at 28 days × 3 to accumulate volume, or
- Re-run at clicks ≥ 10 / impressions ≥ 200 and accept that rows will sit inside the
  noise band. If you choose this, the report must state the lowered thresholds.
```

## Edge cases

- **No comparison period.** Lists 1 and 3 both require it and cannot run. Report list 2 only, and say the other two are `not run — no comparison period`.
- **Periods of unequal length.** Refuse the comparison. A 28-day period against a 30-day one produces click deltas that are mostly calendar.
- **Export empty, or zero clicks in both periods.** Report `no recorded activity` and stop. Do not lower thresholds to find something.
- **Nothing clears any threshold.** Report three zeros with the near-misses and their counts, as above. Never quietly relax a threshold to fill a list.
- **Export too large** — more than 50 000 query × page rows. Aggregate to page level for list 1, then run lists 2 and 3 on the top 5 000 rows by impressions, and say exactly that at the top. Do not sample randomly; the lists are about the head.
- **No weekly rows.** List 3 is `not run — weekly rows required`.
- **No brand list.** Run lists 2 and 3, mark them `brand terms not excluded`, and flag any query containing the site's domain word as a likely brand term.
- **No page titles supplied.** List 2 still runs. Write `Title: (not stated)` rather than fetching or inventing one.
- **A band with under 1 000 impressions site-wide.** It has no usable average. Skip its queries in list 2 and say which band was skipped.
- **A migration or template change inside a period.** State it at the top. Every row is then a candidate for that cause, and the report says so rather than diagnosing page by page.
- **Search Console anonymised queries.** The export drops low-volume queries entirely, so page-level clicks will exceed the sum of their queries. Do not reconcile the two; note the gap and use page-level figures for list 1.

## Stop and hand back

This skill produces three lists. It does not edit a page, and nothing in its output ships
on its own authority. Stop means: hand the list, the counts and the evidence to the named
owner and let them decide. Each of these halts the audit for that row.

- **Any edit to a live page** — a `<title>`, a meta description, a heading, an internal
  link, a template. List 2 exists to say which titles do not match intent; it does not
  rewrite them. A title is customer-facing copy that reaches every searcher with no further
  review, and a bad rewrite costs clicks that will not show up in an export for four weeks.
  Hand the rows to whoever owns the page content or the template.
- **Any consolidation, redirect, canonical or `noindex` arising from list 3.** The list
  names a winner; acting on it means demoting or removing a URL that currently earns
  clicks, and that is not reversible on the same timescale — recovery after a redirect runs
  in weeks, not days. Hand both URLs, both impression shares, the swap evidence and the
  named winner to whoever owns the site's URL structure. Do not write the redirect rule.
- **A URL in the export that should not be indexed** — a staging host, an internal tool, a
  customer-specific page, a checkout or account URL, anything that should sit behind auth.
  That is an exposure, not a ranking finding. Drop the row from the lists, report it to
  whoever owns the site the same day, and say plainly that it is out of this skill's scope.
- **A losing page that is a pricing, legal, or regulated-claims page** — terms, refunds,
  shipping promises, an advertised price, a medical or financial claim. A title or
  description rewrite there changes a published claim. Report the row and route it to
  whoever signs those off. Do not propose replacement wording.
- **A request to lower the thresholds to fill an empty list.** Do not lower them yourself.
  Report the three zeros with their near-misses, present the two options from the small-site
  example, and let the person who asked pick one and own it. Whichever they pick, the
  report states the thresholds actually used.
- **A request to attribute a movement to an algorithm update, or to rank this site against a
  competitor or a published CTR-by-position curve.** The export supports none of the three.
  Say so, say what data would (a rank tracker with a dated index, a competitor's own
  Console), and produce nothing on the current data.
- **A migration, template change or index change inside either period.** Every row is then a
  candidate for that cause, and page-by-page diagnosis is guesswork. State it at the top,
  hand the affected rows to whoever ran the change to rule it out, and do not let anyone
  rewrite a page off this report until they have.

## License
MIT
