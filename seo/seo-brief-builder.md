---
name: seo-brief-builder
owner: rankcraft
category: SEO
description: Builds content briefs from a keyword — search intent, outline, entities to cover, and the questions the top ten never answer.
version: v3
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/rankcraft/seo-brief-builder
raw: https://emdly.com/raw/rankcraft/seo-brief-builder.md
install: npx @emdly/cli add rankcraft/seo-brief-builder
---

# SEO brief builder

A brief is a promise to the writer: cover this, answer these, and the page will deserve to rank. This skill writes that promise from evidence, not from a keyword tool's guess. The output is for one writer about to spend a day on one page, and it must be true of every line in it that the writer can trace it to a result in the supplied SERP or to a question in the supplied related-query data. An outline item that came from the brief-writer's sense of what a page should contain is the thing this skill exists to remove.

## When to use
- For each target keyword before a writer starts.
- To refresh an existing page that stopped ranking, with the current SERP rather than the one it was written for.
- To decide whether a new page should exist at all, when the site may already rank for the query. That decision comes before the brief; see the cannibalization rule.
- To arbitrate between two writers or two teams who both want the query.
- Not for: a page with no search demand, a brand term, or a query the SERP shows to be dominated by a result type the site cannot produce (a store panel, a video carousel, an official source). Say so and stop rather than briefing an unwinnable page.

## Input

Supply all four. Say in one line which you were given.

1. **The target query**, exactly as searched.
2. **The SERP snapshot** — top 10 titles, URLs, **and their H2/H3 headings**. Headings are required, not preferred. Step 3 builds the outline from the union of them, and step 4 reads shared entities out of them. Without headings the outline degrades to a guess dressed as evidence; see Edge cases.
3. **Related queries and "People also ask"** — the raw list.
4. **The site's own existing pages on the topic** — URLs and their target queries, enough to run the cannibalization check.

## Process

1. **Intent.** Classify from the SERP, not from the words in the query. Count what the top ten *do*: compare, teach, sell, list, or answer in one line. Report the count ("7/10 compare"). If no type reaches 6/10, call it mixed, say which type the site can produce, and say what it is giving up.
2. **Angle.** One sentence on how this page differs from the top three. It must be one of these five, named: **first-hand data** the page produced; **a tool or calculator**; **a decision framework** that resolves to a recommendation; **a named population** the top results do not address (a role, a company size, a jurisdiction, a skill level); **currency**, where the top three are demonstrably out of date and the topic changes. Anything outside these five is not an angle. "More comprehensive", "better written", "more in-depth", "higher quality" are refusals to answer the question.
3. **Outline.** H2/H3 skeleton from the union of top-10 headings, deduplicated, ordered by the searcher's journey rather than by frequency. Tag each heading: **table stakes** where 7 or more of the ten have it, **differentiator** where 1 to 3 have it, **contested** where 4 to 6 do. `[house rule]` — the 7 / 4–6 / 1–3 split is a convention for reading the same distribution consistently, not a measured threshold. Report the counts next to each heading so the writer can disagree with the bands.
4. **Entities and terms** the top results share — the things the page must mention to be understood as about this topic. Take terms appearing in 5 or more of the ten. Plain list. No density targets, no counts to hit.
5. **Gaps.** Questions from PAA and related queries that none of the top ten answers in a heading. These are the section headings that win. If a gap question is answered by only one result, it is a differentiator, not a gap; say which.
6. **Internal links.** Existing site pages to link from and to, taken from the supplied page list. Name the URL and the reason. If no site pages are supplied, write `no site pages supplied` rather than inventing plausible URLs.
7. **Specs.** Title, meta description, and length, under the sourcing rules below.

## Rules

- **Never prescribe keyword counts or density.** Entities are a checklist, not a quota.
- **No cannibalization.** Run this before writing anything else. If the site already has a page targeting this query, or ranking for it, say so first, and recommend updating that page instead of briefing a new one. Producing a brief anyway, with a note, is how two pages end up splitting one query.
- **Every outline item traces to the SERP or the gap list.** Each heading carries its count or the word `gap`. A heading with neither is filler and comes out.
- **Title length.** Google publishes no character limit and rewrites titles it dislikes. Truncation in the results page is by **rendered pixel width** — roughly 600px on desktop — not by character count. `≤ 60 characters` is a proxy that holds for average-width text and fails early for capitals and wide letters (W, M) and late for narrow ones. Treat 60 as a check, not a rule: write the title, then say whether it is at risk of truncation and why. Titles with the query early survive rewriting more often.
- **Meta description length.** Same mechanism. `≤ 155 characters` approximates a pixel-width cut of roughly 920px on desktop and less on mobile, and Google frequently ignores the supplied description and generates its own from the page. Write it for the click, front-load the distinguishing clause, and treat the number as a proxy.
- **Target length.** There is no defensible formula. `median ± 30%` is invented and this skill does not use it. Report the top-10 word counts as a distribution — minimum, median, maximum, and any obvious cluster — and let the writer read it. Word count is not a ranking factor; the distribution tells the writer what depth the query's readers currently accept. If word counts are not in the input, say `word counts not supplied` and give no range.

> Thresholds above are defaults; report the thresholds you used.

## Output format

### Example 1 — the normal case, fully filled

```
## Brief: "waterproof trail running shoes"
**Cannibalization check:** ran against 4 supplied site pages. No existing page targets this
query. /shoes/trail is a category page ranking for "trail running shoes" (different intent,
transactional). Proceed.

**Intent:** commercial-investigation, 7/10 (7 compare models, 2 sell a single model,
1 teaches). The site can produce the comparison.
**Angle:** first-hand data — tested in rain, with a measured drying-time table. (Type:
first-hand data. The top three cite manufacturer claims only.)

**Title:** Waterproof Trail Running Shoes: 9 Pairs Tested in Rain (2026)
  61 chars. Query in the first four words. Capital-heavy, and it opens with a "W"; at 61
  chars this is over the ~600px desktop cut and will likely truncate around "(2026)".
  Acceptable — the distinguishing words are before the risk point. Cutting to 60 would not
  save it; the pixels are the constraint, not the count.
**Meta:** We ran 9 waterproof trail shoes through 40 km of rain, then measured how long each
took to dry. Membranes, weights and the two we would not buy.
  144 chars. Google may replace it with its own. The distinguishing clause ("measured how
  long each took to dry") closes at char 94, so it survives a mobile cut.

### Outline
- H2 How we tested (differentiator — 2/10)
- H2 Do you actually need waterproof? (gap — PAA "are waterproof trail shoes worth it",
  no top-10 heading answers it)
- H2 The 9 shoes, compared (table stakes — 9/10)
  - H3 Salomon Speedcross 6 GTX (weight · drop · membrane · drying time)
  - H3 Hoka Speedgoat 6 GTX (same four)
  - H3 Inov-8 Roclite G 315 GTX (same four)
  - H3 Brooks Cascadia 18 GTX (same four)
  - H3 Altra Lone Peak 9 (same four)
  - H3 Saucony Peregrine 15 GTX (same four)
  - H3 La Sportiva Ultra Raptor III GTX (same four)
  - H3 Nnormal Kjerag (same four)
  - H3 Topo Athletic Terraventure 5 (same four)
- H2 Gore-Tex vs eVent vs proprietary membranes (contested — 5/10)
- H2 Drying times, measured (gap — related query "how long do gore-tex shoes take to dry",
  answered by 1/10, so differentiator-adjacent; we own it with data)
- H2 When to choose a non-waterproof shoe instead (differentiator — 3/10)
- H2 Sizing and sock pairing (table stakes — 7/10)
- H2 Care and re-proofing (table stakes — 8/10)

**Entities:** Gore-Tex, eVent, drop (mm), lug depth, Vibram Megagrip, breathability, gaiter
attachment, DWR, stack height. (Each appears in ≥ 5 of the top 10.)
**Links:** from /guides/trail-running-basics (has an unlinked "waterproofing" paragraph) ·
to /shoes/trail (category) · to /reviews/salomon-speedcross-6 (existing review, deepest H3)
**Length:** top-10 word counts — min 1 190, median 2 400, max 5 800. Two clusters: five
results at 1 800–2 600, three at 4 000+. The cluster to match is the first unless the
per-shoe H3s run long. No target given; this is a distribution, not a quota.
```

### Example 2 — the cannibalization case, which ends the brief

```
## Brief: "how to calculate customer acquisition cost"
**Cannibalization check:** ran against 6 supplied site pages. STOP — the site already
targets this query.

/blog/cac-formula — target query "how to calculate customer acquisition cost" (exact match),
published 2023-04, currently ranking position 12 for the query per the supplied snapshot
(it appears in the top 10 for the related query "cac formula" at position 6).

**Recommendation: update /blog/cac-formula. Do not brief a new page.**

A second page on this query splits internal links and the site's own topical signal between
two URLs, and Google will pick one of them for you. What it picks is usually the older one,
which is the page you were trying to replace.

What to change on /blog/cac-formula, from the same SERP evidence:
- Intent: 6/10 teach with a worked calculation, 3/10 offer a calculator, 1/10 sells. The
  existing page teaches but shows no worked example. Add one.
- Missing table stakes: "What to include in CAC" (8/10 have it, the page does not);
  "CAC payback period" (7/10); "CAC:LTV ratio" (7/10).
- Gap worth taking: "does CAC include existing customers' expansion spend" — PAA, answered
  by 0/10.
- Angle for the refresh: decision framework — which costs to include, resolved to a rule
  rather than a list. No top-three result resolves it.
- Title: currently "CAC Formula | Acme" (18 chars, brand suffix, query not in it). Rewrite
  with the query early.
- Length: top-10 min 900, median 1 700, max 3 100. Existing page is 1 100.

**Empty sections for this brief:** no new outline, no new title, no new meta, no internal
link plan. A refresh brief for the existing URL is a separate output; ask for it if that
is the decision.
```

## Edge cases

- **Top-10 headings not supplied**, only titles and URLs. The method collapses. Steps 3, 4 and the table-stakes counts all read the headings; without them the outline is invention. Do not build one from the titles. Return the intent classification, the gap list from PAA, and the cannibalization check — all three survive — and write `outline not produced: top-10 headings not supplied` with what to go and collect. Never emit an outline whose tags say "table stakes" when nothing was counted.
- **Fewer than 10 results supplied.** Report the number and scale the bands to it, saying so ("table stakes: 5 of the 6 supplied"). Below 4 results, do not band at all; list the headings and say the distribution is too small to read.
- **The SERP is mostly not organic pages** — a store panel, a video carousel, a forum block, an official government or medical source at the top. Say what fraction of the visible page an article can occupy, and say plainly if it is not worth briefing. This is more useful than a brief.
- **No related queries or PAA supplied.** The gap list, step 5, cannot be produced. Say `no gap list: related queries not supplied`, and mark the outline as table stakes and differentiators only. Do not label a heading `gap` on the strength of it being absent from the top ten; absence is not demand.
- **No site pages supplied.** The cannibalization rule cannot run. This is a blocking gap, not a cosmetic one: write `cannibalization check not run — no site pages supplied` at the top of the brief in place of the check, and say the brief must not be actioned until someone confirms the site does not already target the query.
- **Intent is genuinely split**, no type reaching 6/10. Say the split, name the one type the site can win, and list what the page gives up by choosing it. Do not brief a page that serves two intents; it serves neither.
- **The query is ambiguous** — two different meanings in the top ten (a brand and a common noun, two industries). Say which meaning the brief covers and how many results belong to each. If the split is near even, ask before writing.
- **Word counts not supplied** with the SERP. Omit the length section entirely and say so. Do not estimate word counts from titles.
- **Query has no measurable demand** in the supplied data, or the SERP is thin and unrelated. Say the SERP does not establish an intent and the brief has no evidence base. Recommend against the page.
- **The topic is medical, financial, or legal advice.** The outline and the entity list still hold, but say in the brief that the page needs a named qualified reviewer and that the writer must not resolve a decision framework in these areas without one. Name the requirement in the brief; do not bury it.
- **The SERP snapshot is stale** — more than about a quarter old, or the input does not say when it was captured. Say the capture date, or say it is unknown, and mark the intent classification and the counts as provisional. Refreshing a page against last year's SERP is how a page gets rewritten toward what already stopped working.

## License
MIT
