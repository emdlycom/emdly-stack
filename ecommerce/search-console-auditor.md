---
name: search-console-auditor
owner: shopmetric
category: Ecommerce
description: Audits Google Search Console data — pages losing clicks, queries with impressions but no CTR, and product pages cannibalizing each other.
version: v2
license: MIT
updated: 2026-08-19
recommended: false
security_checked: true
url: https://emdly.com/skills/shopmetric/search-console-auditor
raw: https://emdly.com/raw/shopmetric/search-console-auditor.md
install: npx @emdly/cli add shopmetric/search-console-auditor
---

# Search Console auditor

Search Console has the answers; it just does not sort them. This skill produces the three lists that are worth acting on and ignores the rest.

## When to use
- Monthly, on a 28-day export vs. the previous 28 days (queries × pages, with clicks, impressions, CTR, position).
- After a migration or a template change.

## The three lists

**1. Losing pages.** Pages whose clicks fell ≥ 20% and ≥ 50 clicks in absolute terms. For each: did impressions fall (demand or indexing) or did CTR/position fall (the page or the SERP changed)? Say which.

**2. Free clicks.** Queries with ≥ 1 000 impressions, position ≤ 8 and CTR under half the site's average for that position band. These are titles and descriptions that do not match the intent. Quote the current title.

**3. Cannibalization.** A query where two URLs each get ≥ 10% of its impressions in both periods, and the ranking URL swaps between weeks. Name both URLs and which one should win (the one with the higher CTR, unless it is a category page fighting a product page — then the category).

## Rules
- Thresholds above are defaults; report the thresholds you used.
- Position is an average and moves with impressions — never call a 0.3 change a "drop".
- Brand queries are excluded from lists 2 and 3 (they behave differently); list them separately if they moved.
- Do not suggest content to write. This skill finds problems on existing pages.

## Output format
```
## Losing pages (4)
- /shoes/trail-runners — clicks 1 240 → 810 (−35%). Impressions flat, position 4.1 → 7.8: the page lost rank, demand did not move. Check: 2 new competitors in top 5, page unchanged since March.

## Free clicks (6)
- "waterproof trail running shoes" — 14 200 impr., pos. 5.2, CTR 1.1% (band avg 4.8%). Title: "Trail Runners | Shop" → says nothing about waterproof.

## Cannibalization (2)
- "merino socks" — /socks/merino (cat.) vs /socks/merino-hiker-2 (product); swapped 3× in 4 weeks. Winner: category; add the product to its top grid and link back from the product.
```

## License
MIT
