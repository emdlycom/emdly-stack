---
name: bestseller-rank-report
owner: launifycorp
category: Ecommerce
description: This skill owns a single deliverable: a defensible ranked list of topselling SKUs for a named period, ranked by both units sold and net revenue, with movement since the prior comparable period and the...
version: v1
license: MIT
updated: 2026-09-24
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/bestseller-rank-report
raw: https://emdly.com/raw/launifycorp/bestseller-rank-report.md
install: npx @emdly/cli add launifycorp/bestseller-rank-report
---

# Bestseller Ranking Report

This skill owns a single deliverable: a defensible ranked list of top-selling SKUs for a named period, ranked by both units sold and net revenue, with movement since the prior comparable period and the concentration of sales at the head of the list. The output is the artifact merchandising, buying and marketing teams use to decide reorders, homepage slots and paid budget. It ends where decisions begin — you rank and explain, you do not set the reorder quantity.

The judgement call that separates a good report from a mediocre one is the reconciliation between units and revenue. Ranking by units alone promotes cheap accessories and bundle fillers; ranking by revenue alone promotes one-off high-ticket items that will never repeat. A good report shows both rankings side by side, names every SKU whose two ranks diverge by 10 places or more, and states which ranking the reader should act on for the specific decision at hand. The second judgement call is what counts as a sale: returns, cancellations, gift cards, shipping revenue and internal test orders all move the numbers, and you declare your treatment of each in the header block before the first data row is printed.

## When to use

- A stakeholder asks for "our top 20 bestsellers last month" or "nejprodávanější produkty" for a period with a defined start and end date.
- A monthly or quarterly commercial review needs a bestseller slide with movement versus the prior period.
- A buyer asks which SKUs to reorder ahead of a season and wants the sales side of that decision quantified.
- An export of order lines, or a platform report (Shopify, Shoptet, WooCommerce, GA4 Item Report), lands on your desk with a request to "turn this into a ranking".
- Marketing needs a shortlist of hero products for a campaign, newsletter or category landing page.
- A category or brand manager wants their subset ranked inside the wider assortment, with the subset's share of total revenue stated.

**Do not use this when:**

- The question is *why* sales moved — that is sales variance analysis; reach for a driver decomposition (price × volume × mix), not a ranking.
- The question is what to reorder and how much — that is demand planning; a ranking is an input, but you need lead times, stock on hand and sell-through rate.
- The question is which products to drop — that is assortment rationalisation; you need margin, stock cover and cannibalisation, and a bestseller list is the wrong end of the distribution.
- The period contains fewer than 300 order lines and no launch or campaign to explain the window — the ranking will not survive a week; widen the period first (see Judgement calls).

## Inputs

| Input | Required | If missing |
|---|---|---|
| Order line export (SKU, quantity, line net amount, order date, order status) | Yes | Stop. Request the export with these five fields minimum. No substitute produces a rankable dataset. |
| Period definition (start date, end date, timezone, date basis: order date vs ship date) | Yes | Default to full calendar months, order date, store timezone; state the default in the header and flag it for confirmation in the first caveat line. |
| Comparison period | Recommended | Default to the immediately prior period of equal calendar length; add prior-year same period if 13+ months of history exist. With no history, print movement columns as "prior period only" and omit NEW/RISER/FALLER flags entirely. |
| Product master (SKU → product name, category, brand, variant attributes) | Recommended | Rank on raw SKU codes, mark names as `UNMAPPED`, and list unmapped SKUs with their unit and revenue totals in an appendix. Do not guess product names from SKU strings. |
| Returns / refunds data keyed to original order line | Recommended | Rank on gross sales, label every figure "gross, returns not deducted", and state the store-level return rate if known from any other source. |
| Cost or margin per SKU | No | Omit the margin column entirely rather than estimating. Do not apply a blanket margin percentage. |
| Discount allocation at line level | No | Use gross line value, rename the column `Revenue (pre-discount)`, and state the store-wide discount rate for the period. |

The non-negotiable core is order lines with SKU, quantity, net amount and date; everything else degrades into a labelled caveat. Put a "Data basis" block at the top that names what is included, what is excluded, and which figures are gross rather than net. A ranking on gross sales with the treatment declared is useful; a ranking with an undeclared basis is worse than no ranking, because it will be quoted back at you in six months.

## Method

1. **Fix the period and the date basis before touching the data.** Write down start timestamp, end timestamp, timezone and whether you are using order creation date, payment date or ship date. Order date is the default because it matches marketing attribution; ship date matches logistics and warehouse planning.
   - If the request is "last month" and today is the 3rd, use the complete prior calendar month, not a trailing 30 days. If the requester says "last 30 days", use a trailing window ending yesterday at 23:59:59 in store timezone and say so explicitly. Never mix the two conventions in one report.
   - Record the day count of both periods. If they differ by more than 5% (e.g. 29 vs 31 days = 6.9%), state the difference in the caveats and do not day-adjust the headline figures; day-adjusted units may appear as a second caveat line only.

2. **Define the sale and filter the order lines.** Decide inclusion per status and apply the identical filter to the ranking period and the comparison period. Print the filter as a one-line rule in the header.
   - Include: paid, fulfilled, partially fulfilled. Exclude: cancelled, unpaid, draft, test orders, staff orders where a staff discount code is identifiable, gift card line items, shipping and handling lines, tax lines.
   - Compute cancelled units as a share of gross units. If it exceeds 5%, print the figure in the header — it signals a payment gateway or stock-accuracy problem the reader will want to know about. Below 5%, report it in the caveats.
   - Identify test orders by: order email domain matching the store's own domain, order total of 0, or orders flagged `test` by the platform. Count them and state the count.

3. **Normalise SKU granularity and decide the roll-up level.** Choose variant-level (each size/colour separate) or product-level (variants summed) and state which in the header.
   - Default to product-level when N ≤ 20 and the request mentions marketing, homepage, newsletter or a meeting; default to variant-level when the request mentions stock, sizes, colours, reorder or purchase order.
   - If, at variant level, more than 3 of the top 10 rows share one parent product ID, roll that product up to product level, re-rank, and note the change in one line — otherwise the list becomes a single-product report.
   - Strip trailing bundle and channel suffixes (`-BUNDLE`, `-FBA`, `-GIFT`) into the parent SKU only when the product master confirms the mapping. Never merge on string similarity.

4. **Compute the two ranking metrics plus the supporting columns.** For each SKU compute: units = sum of quantity; revenue = sum of line net amount after line-level discounts, before shipping and tax; orders = count of distinct orders containing the SKU; ASP = revenue ÷ units; attach rate = orders containing SKU ÷ total orders in period.
   - Revenue must be net of discounts. If discount allocation is unavailable at line level, use gross line value and relabel as in Inputs.
   - Exclude any SKU with net units ≤ 0 (returns equalling or exceeding sales) from both rankings and list it under "Net-negative SKUs" with its unit and revenue balance.
   - Round only at presentation time. Compute ranks on unrounded values; break ties on revenue first, then SKU code ascending, so the ordering is reproducible.

5. **Produce both rankings and quantify divergence.** Rank descending by units and separately by revenue. Compute `rank_delta = rank_by_units − rank_by_revenue` for every SKU in the union of the two top-N lists.
   - Flag any SKU with `|rank_delta| ≥ 10` as divergent and write one sentence naming the cause: high units + low revenue means a cheap consumable, add-on or bundle filler (support it with the attach rate); low units + high revenue means a high-ticket or low-repeat item (support it with its share of period revenue).
   - Cap the divergence section at the 5 largest `|rank_delta|` values; the rest go to the appendix.
   - If more than 40% of the union top 20 is divergent, the assortment spans two price tiers — split the report into price bands (e.g. under 500 CZK and 500 CZK and above) and rank inside each, rather than forcing one list.

6. **Attach the comparison period and classify movement.** For each SKU in the current top N, pull prior-period units and revenue, then compute rank change and percentage change in units.
   - Classify in this order: `NEW` if prior-period units = 0; `LOW BASE` if prior-period units are 1–9, with percentage suppressed as "n/a (low base)"; `RISER` if units grew ≥ 30%; `FALLER` if units fell ≥ 30%; `STABLE` otherwise.
   - If the comparison period contains a known distortion — Black Friday, a site outage longer than 4 hours, a stockout of a top-10 SKU, a price change above 10% — name it in one line rather than letting the reader infer a trend that is a calendar artefact.
   - Check for cannibalisation before writing commentary: if a FALLER and a RISER/NEW share a parent category and their unit movements offset within ±25%, say so.

7. **Measure concentration.** Compute the share of total period revenue held by the top 10 SKUs and by the top 20, and the number of SKUs needed to reach 50% of cumulative revenue.
   - Assess against fixed thresholds: top 10 ≥ 50% of revenue is **high** concentration and a supply-risk flag; 20–50% is **moderate**; below 20% is a **long-tail** assortment where a bestseller list drives little of the business, and you say so in the read-out.
   - When concentration is high, name the single largest SKU's revenue share and state the exposure in plain terms ("a stockout removes X% of monthly revenue").

8. **Write the read-out, then sanity-check totals.** Add three to five bullets naming what changed and what the reader should look at next. Before shipping, reconcile ranked revenue + unranked tail against the period revenue total from the source system.
   - Tolerance: 0.5%. Above that, isolate the gap in this order — shipping lines, tax treatment, status filter, currency conversion, unmapped SKUs — before publishing. Do not plug the difference.
   - Print the reconciliation line with both totals and the variance percentage to two decimals, whether it passes or fails.

## Judgement calls

**Units vs revenue as the lead ranking** — Lead with units when the reader is buying stock, planning warehouse space, or writing a "customer favourites" page; lead with revenue when the reader is allocating paid media budget or reporting to finance. When the request is ambiguous, lead with revenue, show units adjacent, and name in the read-out the SKUs that change position. What tips the balance: compute the ASP spread across the top 20 (highest ASP ÷ lowest ASP). Above 5×, revenue must lead or the units ranking will mislead; below 2×, the two rankings will largely agree and either order works.

**Variant-level vs product-level detail** — Variant-level whenever the answer must survive contact with a purchase order; product-level whenever the answer will be read aloud in a meeting. What tips the balance is variant count: if the median top-20 product has more than 4 active variants, product-level is the only readable option, with a variant breakdown for the top 3 products in an appendix. If the top-20 products average fewer than 2 variants, report variant-level regardless of audience — the roll-up buys nothing.

**Short period vs statistically sound one** — A weekly bestseller list is legitimate for reactive merchandising but unstable: any SKU below 20 units in the period can swing 10 rank places on normal variance. If the period yields fewer than 300 order lines, or the rank-20 SKU has fewer than 20 units, cap the ranking at top 10 and state the instability in the caveats. What tips the balance: a launch or campaign inside the window makes the short period the point — keep it, annotate the launch date, and mark affected SKUs `NEW` rather than comparing.

**Including or excluding returns** — Deduct returns whenever return data is keyed to original order lines and the store-level return rate exceeds 10%, typical of apparel and footwear; report gross below that threshold for speed and say so. What tips the balance is category skew: if any category's return rate is more than double the store average, gross ranking systematically over-promotes it — deduct returns for that category or annotate every affected row with its category return rate.

## Rules

- Never invent a product name, category or price. Unmapped SKUs stay as raw codes with an `UNMAPPED` label and appear in the appendix with units and revenue.
- Never blend date bases within one report. Order date throughout, or ship date throughout.
- Never show a percentage change on a prior-period base below 10 units; print "n/a (low base)" and flag the row `LOW BASE`.
- Never deduct returns in one period and not the other; the status filter, the date basis and the returns treatment must be identical across both periods.
- State the currency and whether figures include or exclude VAT in the header of every table.
- Round revenue to whole currency units, ASP to two decimals, shares and percentage changes to one decimal place. Rank deltas are integers. Do not present more precision than the source supports.
- Mark any figure derived from an incomplete feed with `†` and footnote what is missing.
- Reorder quantities, delisting, pricing changes and media budget allocation belong to the human. You may write "the top 3 SKUs carry 25.6% of revenue and one is flagged FALLER"; you may not write "order 400 units" or "cut spend on filters".
- If the step 8 reconciliation fails by more than 0.5%, publish with an explicit `UNRECONCILED VARIANCE` line naming the amount and the suspected source. Never silently adjust a number to close the gap.
- Cap the main ranking at 20 rows unless a longer list is explicitly requested; ranks 21+ go to an appendix.
- Every SKU appearing in the read-out must appear in one of the two tables with its numbers visible.

## Output format

```
# Bestseller Ranking — [Period label]

**Data basis:** [date basis] | [timezone] | Statuses included: [list] | Excluded: [list]
**Currency:** [CCY], [incl./excl. VAT] | **Returns:** [deducted / not deducted]
**Comparison period:** [dates] | **Roll-up level:** [product / variant]
**Total period revenue:** [X] | **Total units:** [Y] | **Orders:** [Z]
**Reconciliation:** ranked + tail = [X] vs source [X] ([n.nn]% variance)

## Top [N] by revenue
| # | SKU | Product | Revenue | Units | ASP | Orders | Rev Δ vs prior | Rank Δ | Flag |
|---|-----|---------|---------|-------|-----|--------|----------------|--------|------|
| 1 | | | | | | | | | |

## Top [N] by units
| # | SKU | Product | Units | Revenue | ASP | Units Δ vs prior | Rank Δ | Flag |
|---|-----|---------|-------|---------|-----|------------------|--------|------|
| 1 | | | | | | | | |

## Divergent SKUs (|rank delta| ≥ 10, largest 5)
- [SKU] — units rank [n], revenue rank [m], delta [d] — [one-sentence reason with attach rate or revenue share]

## Concentration
- Top 10 = [x.x]% of period revenue; top 20 = [y.y]%
- [n] SKUs reach 50% of revenue
- Assessment: [high / moderate / long-tail] concentration

## Movement
- NEW entrants: [SKU list]
- RISERS (≥30% units): [SKU list]
- FALLERS (≥30% units): [SKU list]
- LOW BASE (prior units 1–9): [SKU list]

## Read-out
- [3-5 bullets: what changed, what it means, what the reader should look at next]

## Caveats and exclusions
- [each assumption, missing field, distortion, day-count difference, unmapped SKU count]
```

Keep the whole deliverable under two pages: two tables of at most 20 rows, five divergence lines, five read-out bullets. Order is fixed — data basis first, revenue ranking second, because the revenue table is what gets quoted. When it runs long, cut in this order: the units table down to top 10, then the movement lists to NEW and FALLERS only, then the appendix. Never cut the data basis block, the reconciliation line, or the caveats.

## Worked example

**Input:** Shoptet order-line export, 4,118 paid lines, 1–31 March 2024, CZK excl. VAT, product master supplied (6 SKUs unmapped), no returns file. Kitchenware store, 2,204 orders in period. Request: "Dej mi nejprodávanější produkty za březen, top 10, a jestli něco vyskočilo."

**Decisions taken before writing:** order date basis (marketing will read this); product level (request is merchandising-flavoured, no mention of stock or sizes); comparison to 1–29 Feb 2024, which is 6.9% shorter — flagged, not adjusted; ASP spread across the top 20 is 6,157 ÷ 70 = 88×, far above 5×, so revenue leads.

**Output:**

```
# Bestseller Ranking — March 2024

Data basis: order date | Europe/Prague | Included: paid, fulfilled, partially fulfilled
             Excluded: cancelled, unpaid, draft, gift cards, shipping lines, tax lines, 4 test orders
Currency: CZK excl. VAT | Returns: NOT deducted (no returns feed) †
Comparison period: 1–29 Feb 2024 (29 days vs 31) | Roll-up: product level
Total period revenue: 2,418,600 | Units: 6,942 | Orders: 2,204
Reconciliation: ranked (1,622,800) + tail (795,800) = 2,418,600 vs source 2,417,900 (0.03% variance)

## Top 10 by revenue
| #  | SKU     | Product                          | Revenue | Units | ASP      | Orders | Rev Δ | Rank Δ | Flag   |
|----|---------|----------------------------------|---------|-------|----------|--------|-------|--------|--------|
| 1  | KV-2200 | Kávovar Aurora Duo               | 289,400 | 47    | 6,157.45 | 47     | +38%  | +2     | RISER  |
| 2  | MX-118  | Mixér Vertex Pro                 | 176,200 | 112   | 1,573.21 | 110    | −6%   | −1     | STABLE |
| 3  | PN-041  | Pánev Granit 28 cm               | 154,800 | 398   | 388.94   | 341    | NEW   | NEW    | NEW    |
| 4  | KV-7700 | Kávovar Aurora Mini              | 131,000 | 39    | 3,358.97 | 39     | −34%  | −2     | FALLER |
| 5  | ZR-310  | Zrnková káva Sunrise 1 kg        | 118,900 | 421   | 282.42   | 318    | +12%  | 0      | STABLE |
| 6  | HR-560  | Hrnec Granit 5 l                 | 96,400  | 214   | 450.47   | 201    | +43%  | +1     | RISER  |
| 7  | NZ-220  | Nůž šéfkuchařský Damask 20 cm    | 88,700  | 131   | 677.10   | 128    | +6%   | −1     | STABLE |
| 8  | RY-045  | Rychlovarná konvice Vertex 1,7 l | 74,300  | 167   | 444.91   | 165    | −1%   | 0      | STABLE |
| 9  | ES-980  | Espresso set Aurora (2 ks)       | 61,500  | 205   | 300.00   | 178    | −4%   | 0      | STABLE |
| 10 | MD-112  | Mlýnek na kávu Aurora Grind      | 58,900  | 84    | 701.19   | 84     | +18%  | +2     | STABLE |

## Top 10 by units
| #  | SKU     | Product                          | Units | Revenue | ASP    | Units Δ        | Rank Δ | Flag     |
|----|---------|----------------------------------|-------|---------|--------|----------------|--------|----------|
| 1  | FL-009  | Filtry papírové 100 ks           | 612   | 42,800  | 69.93  | +9%            | 0      | STABLE   |
| 2  | ZR-310  | Zrnková káva Sunrise 1 kg        | 421   | 118,900 | 282.42 | +12%           | 0      | STABLE   |
| 3  | PN-041  | Pánev Granit 28 cm               | 398   | 154,800 | 388.94 | NEW            | NEW    | NEW      |
| 4  | HR-560  | Hrnec Granit 5 l                 | 214   | 96,400  | 450.47 | +43%           | +2     | RISER    |
| 5  | ES-980  | Espresso set Aurora (2 ks)       | 205   | 61,500  | 300.00 | −4%            | −2     | STABLE   |
| 6  | RY-045  | Rychlovarná konvice Vertex 1,7 l | 167   | 74,300  | 444.91 | −1%            | −1     | STABLE   |
| 7  | UT-330  | Utěrka mikrovlákno 3 ks          | 158   | 11,000  | 69.62  | +159%          | +8     | RISER    |
| 8  | PN-048  | Pánev Granit 24 cm               | 142   | 49,800  | 350.70 | −31%           | −4     | FALLER   |
| 9  | NZ-220  | Nůž šéfkuchařský Damask 20 cm    | 131   | 88,700  | 677.10 | +6%            | 0      | STABLE   |
| 10 | KO-210  | Konvice Aurora french press      | 118   | 39,600  | 335.59 | n/a (low base) | +21    | LOW BASE |

## Divergent SKUs (|rank delta| ≥ 10)
- UT-330 — units rank 7, revenue rank 28, delta −21 — 158 units of a 70 CZK cloth pack; 0.5% of revenue, present in 7.1% of orders as a basket filler.
- KV-2200 — units rank 19, revenue rank 1, delta +18 — 47 machines carry 12.0% of month revenue; one unit is worth 88 packs of filters.
- KV-7700 — units rank 21, revenue rank 4, delta +17 — same pattern, 39 units at 3,359 CZK ASP.
- FL-009 — units rank 1, revenue rank 13, delta −12 — consumable attached to coffee machines; attach rate 18.0% of orders (397 of 2,204).
- SD-400 — units rank 22, revenue rank 11, delta +11 — 38 knife sets at 1,426 CZK ASP, 2.2% of revenue.

Divergent SKUs are 5 of the 22 in the union top 20 (22.7%), below the 40% price-band split threshold, so one combined list stands.

## Concentration
- Top 10 = 51.7% of period revenue; top 20 = 67.1%
- 10 SKUs reach 50% of revenue
- Assessment: high concentration (top 10 ≥ 50%) — supply-risk flag

## Movement
- NEW entrants: PN-041
- RISERS (≥30% units): KV-2200 (+38%), HR-560 (+43%), UT-330 (+159%), SD-400 (+31%)
- FALLERS (≥30% units): KV-7700 (−34%), PN-048 (−31%)
- LOW BASE (prior units 1–9): KO-210 (8 → 118)

## Read-out
- Concentration crossed the 50% line this month: the top 10 hold 51.7% of revenue and Kávovar Aurora Duo alone holds 12.0%. A stockout on one SKU removes roughly an eighth of monthly revenue.
- Pánev Granit 28 cm entered at revenue rank 3 after its 4 March launch — 398 units in 27 days, no prior-period base. Pánev Granit 24 cm fell 31% (205 → 142 units) over the same window; the offset is within 25%, so read this as cannibalisation inside the Granit pan line, not category growth of 256 units.
- Aurora Mini fell 34% in units in the month Aurora Duo rose 38% (34 → 47). Combined Aurora machine units are 93 vs 93 in February — flat category, shifted mix toward the higher-ASP model.
- Filtry papírové leads units at 612 but ranks 13 on revenue (0.5× the revenue of the rank-10 item). The ASP spread across the top 20 is 88×; the units table should not be used for media allocation this month.
- Konvice Aurora french press moved from 8 to 118 units. The prior base is below 10, so no percentage is shown — confirm whether this is a real shift or a February listing gap before treating it as a trend.

## Caveats and exclusions
- † All figures gross of returns; no returns feed supplied. Store return rate unknown, so the 10% deduction threshold could not be tested.
- February is 29 days vs March 31 (6.9% fewer days). Units comparisons are not day-adjusted. Day-adjusted, KV-2200 grew 29% rather than 38%, and PN-048 fell 26% rather than 31% — the FALLER classification for PN-048 is borderline on that basis.
- Cancelled orders were 3.1% of gross units (below the 5% header-flag threshold). 4 test orders excluded (store-domain email, total 0).
- 6 SKUs (41 units, 8,900 CZK, 0.4% of revenue) are unmapped in the product master; all sit outside the top 20 and are listed in the appendix as raw codes.
- Discounts were allocated at line level, so revenue is net of discounts; store-wide discount rate 7.4%.
```

## Quality bar

- [ ] Period, timezone, date basis, currency, VAT treatment, roll-up level and status filter all appear in the header before any data row.
- [ ] Both rankings are present, and every SKU with `|rank delta| ≥ 10` has a one-sentence explanation containing either an attach rate or a revenue share.
- [ ] Ranked revenue plus tail reconciles to the source total within 0.5%, and both totals plus the variance percentage are printed.
- [ ] Every percentage change rests on a prior-period base of at least 10 units; all others read "n/a (low base)" and carry the `LOW BASE` flag.
- [ ] Top 10 share, top 20 share and SKUs-to-50% are all computed, and the assessment word (high / moderate / long-tail) matches the stated thresholds.
- [ ] No product name, category or price appears that is not in the supplied product master; unmapped SKUs are labelled `UNMAPPED` and counted in the caveats.
- [ ] The read-out contains no reorder quantity, price change, delisting or budget instruction.
- [ ] Every SKU named in the read-out appears in one of the two tables.
- [ ] Day counts of both periods are stated whenever they differ by more than 5%.
- [ ] The main ranking is 20 rows or fewer, and the divergence section is 5 lines or fewer.

## Failure modes

**A single product floods the top 10** — variant-level roll-up on a product with many sizes or colours — check whether more than 3 of the top 10 rows share a parent product ID; if so, roll up to product level, re-rank, and note the change in one line.

**Revenue total does not match the platform dashboard** — shipping revenue, tax, or cancelled orders included on one side only — run the step 8 reconciliation before writing any commentary, and isolate the gap by re-running the total with shipping lines excluded, then with tax excluded, then with the status filter relaxed. Fix the filter, never the number.

**A "riser" that is a calendar artefact** — comparison period is a different length or contains a campaign — check day counts and known promo dates before classifying movement; if day counts differ by more than 5%, print both raw and day-adjusted changes in the caveats and say which classification is borderline.

**Ranking is unstable week to week** — period too short, top-20 SKUs sitting below 20 units each — check the units of the rank-20 SKU; if under 20 or the period holds fewer than 300 order lines, cap the list at 10 and state the instability in the caveats.

**Cheap consumables dominate and the reader buys the wrong stock** — units ranking presented without the revenue counterpart — compute the ASP spread across the top 20; above 5× the revenue table leads, the divergence section is mandatory, and the read-out must name the consumables explicitly.

**Two products swap sales and the report calls it growth** — a new variant cannibalises its predecessor — before writing commentary, check whether any FALLER and any RISER/NEW share a parent category with unit movements offsetting within ±25%; if so, report the combined category figure alongside the individual rows.

## License

MIT
