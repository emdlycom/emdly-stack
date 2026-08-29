---
name: seo-brief-builder
owner: rankcraft
category: SEO
description: Builds content briefs from a keyword — search intent, outline, entities to cover, and the questions the top ten never answer.
version: v2
license: MIT
updated: 2026-08-24
recommended: false
security_checked: true
url: https://emdly.com/skills/rankcraft/seo-brief-builder
raw: https://emdly.com/raw/rankcraft/seo-brief-builder.md
install: npx @emdly/cli add rankcraft/seo-brief-builder
---

# SEO brief builder

A brief is a promise to the writer: cover this, answer these, and the page will deserve to rank. This skill writes that promise from evidence, not from a keyword tool's guess.

## When to use
- For each target keyword before a writer starts.
- To refresh an existing page that stopped ranking.

## Input
The target query, the SERP snapshot (top 10 titles, URLs, and — ideally — their headings), related queries / "People also ask", and the site's own existing pages on the topic.

## Process
1. **Intent.** Classify from the SERP, not from the words: what do the top results *do* (compare, teach, sell, list)? If the top ten mix intents, say which one the site can win.
2. **Angle.** One sentence on how this page will differ from the top three — not "more comprehensive"; a real angle (first-hand data, a tool, a decision framework).
3. **Outline.** H2/H3 skeleton built from the union of top-10 headings, deduplicated, ordered by the searcher's journey. Mark which headings every top result has (table stakes) and which only one or two have (differentiators).
4. **Entities and terms** the top results share — things the page must mention to be understood as about this topic. Plain list, no density targets.
5. **Gaps.** Questions from PAA and related queries that none of the top ten answers. These are the section headings that win.
6. **Internal links:** existing site pages to link from and to.
7. **Specs:** target length as a range from the top 10 (median ± 30%), title ≤ 60 chars with the query, meta description ≤ 155.

## Rules
- Never prescribe keyword counts or density. Entities are a checklist, not a quota.
- No cannibalization: if the site already ranks for the query on another URL, say so first and recommend updating that page instead.
- Every outline item traces to the SERP or the gap list — no filler sections.

## Output format
```
## Brief: "waterproof trail running shoes"
**Intent:** commercial-investigation (7/10 results compare models). **Angle:** tested in rain, with a drying-time table.
**Title:** Waterproof Trail Running Shoes: 9 Pairs Tested in Rain (2026)

### Outline
- H2 How we tested (differentiator)
- H2 The 9 shoes, compared (table stakes — 9/10 have it)
  - H3 per shoe: weight · drop · Gore-Tex vs eVent · drying time
- H2 Do you actually need waterproof? (gap — PAA, nobody answers)
…
**Entities:** Gore-Tex, eVent, drop, lug depth, Vibram Megagrip, breathability
**Links:** from /guides/trail-running-basics · to /shoes/trail (category)
**Length:** 1 800–3 200 words
```

## License
MIT
