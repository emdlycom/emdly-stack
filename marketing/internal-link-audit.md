---
name: internal-link-audit
owner: launifycorp
category: Marketing
description: You scan a defined set of blog posts, map the internal links that exist between them, and produce a prioritized list of missing, broken, or wasteful links with exact anchor text and insertion points....
version: v1
license: MIT
updated: 2026-09-15
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/internal-link-audit
raw: https://emdly.com/raw/launifycorp/internal-link-audit.md
install: npx @emdly/cli add launifycorp/internal-link-audit
---

# Internal Link Audit

You scan a defined set of blog posts, map the internal links that exist between them, and produce a prioritized list of missing, broken, or wasteful links with exact anchor text and insertion points. The deliverable you own is a per-post action list a writer or editor can execute without further research.

## When to use

- A blog has 15+ published posts and no one has checked link structure since launch.
- New cluster posts were published and nobody linked the older posts to them.
- Organic traffic is flat on posts that should support a money page or pillar page.
- A site migration, URL slug change, or CMS move happened and links may have broken.
- Before a content refresh sprint, to decide which posts get updated first.

Do not use when:

- The task is external/backlink acquisition or outreach — that is a different job.
- Fewer than 6 posts exist; there is not enough surface area to produce a useful audit.

## Inputs

Before starting, you need:

1. **Post inventory** — a list of URLs or files with, at minimum: URL/slug, title, primary topic or target keyword, publish date.
2. **Access to post body content** — full text or HTML, so you can read existing links and find insertion contexts.
3. **Site structure** — which posts are pillars/hubs, which are supporting posts, and which pages are conversion targets.
4. **Link policy** — max links per post, whether nofollow is used internally, anchor-text conventions.

If something is missing, ask for it explicitly and do not guess:

- No topic/keyword per post → ask, or derive from H1 + first paragraph and mark the derivation as `[inferred]` in the output.
- No pillar designation → ask; if unavailable, treat the post with the broadest topic and most inbound links as the hub and flag the assumption.
- No link policy → default to max 5 internal links per 1,000 words and state the default in the report.
- No body content, only URLs → stop and request content. Do not audit from titles alone.

## Method

1. **Build the inventory table.** One row per post: URL, title, topic, word count, publish date. If two posts share the same primary topic, mark both `CANNIBALIZATION?` and continue.
2. **Extract every internal link.** For each post, list outbound internal links as (source URL, anchor text, target URL, section where it appears). Ignore nav, footer, sidebar, and related-post widgets — audit body links only.
3. **Classify each existing link.** Assign exactly one status: `OK`, `BROKEN` (target not in inventory or returns error), `REDIRECT` (target differs from canonical slug), `WEAK_ANCHOR` (anchor is "here", "this post", "click", or the bare URL), `IRRELEVANT` (target topic unrelated to the surrounding paragraph). Anything not clearly in another class is `OK`.
4. **Count inbound links per post.** Any post with 0 inbound body links is an orphan; any post with 1 is under-linked. Rank by fewest inbound links first.
5. **Score topical proximity.** For each unlinked post pair, decide `STRONG` (same cluster, one is clearly narrower or broader than the other), `MEDIUM` (shares an entity, audience, or use case), or `NONE`. Only `STRONG` and `MEDIUM` pairs become recommendations.
6. **Locate the insertion point.** For every recommended link, find the specific sentence or paragraph in the source post where the target's topic is already mentioned or implied. Quote 10–20 words of that sentence. If no such sentence exists, mark the recommendation `NEEDS_NEW_SENTENCE` and draft one line of copy.
7. **Write the anchor text.** Use 2–6 words containing the target's primary topic. Never reuse the same anchor for two different targets. Never use the target's exact title if it reads unnaturally in the sentence.
8. **Check hub coverage.** Every supporting post must link up to its pillar; every pillar must link down to at least 3 supporting posts. List violations as `HUB_GAP` items.
9. **Enforce link density.** Before finalizing, count existing plus recommended links per post. If the total exceeds the policy limit, drop the lowest-proximity recommendations until it fits and note what was dropped.
10. **Prioritize.** Sort all actions: P1 = broken/redirect fixes and orphan rescues, P2 = hub gaps, P3 = STRONG new links, P4 = MEDIUM new links and weak-anchor rewrites.

## Rules

- Never invent a URL. If a target is not in the supplied inventory, it does not exist for this audit.
- Never recommend a link you cannot place in a specific, quoted sentence or a drafted replacement sentence.
- Never recommend more than 3 new links into a single target from the same source post.
- Never edit post content. You produce recommendations; a human or a separate writing step applies them.
- Never count navigation, breadcrumb, footer, or auto-generated related-posts links as internal links.
- Do not recommend reciprocal links between two posts unless both directions have a distinct, relevant insertion point.
- Respect the link policy limit; if none was given, state the default you applied at the top of the report.
- Mark every inference with `[inferred]`. Unverifiable status (e.g. HTTP checks you cannot run) is `UNVERIFIED`, never `OK`.
- Cap the report at 40 recommendations. If more exist, ship the top 40 by priority and state the total found.
- If the inventory has fewer than 6 posts, stop and report insufficient scope.

## Output format

```
# Internal Link Audit — [Blog name]
Date: [YYYY-MM-DD]
Posts audited: [N]
Link policy applied: [max X links per 1,000 words — source: provided / default]
Total recommendations found: [N] (showing top [N])

## 1. Summary
- Orphan posts (0 inbound): [N]
- Under-linked posts (1 inbound): [N]
- Broken internal links: [N]
- Redirected internal links: [N]
- Weak anchors: [N]
- Hub gaps: [N]

## 2. Post inventory
| # | Title | URL | Topic | Words | Inbound | Outbound | Role |
|---|-------|-----|-------|-------|---------|----------|------|
| 1 | | | | | | | pillar/support |

## 3. Fixes (P1)
### FIX-01 | BROKEN
- Source: [URL]
- Current anchor: "[text]"
- Current target: [URL] — status: BROKEN / REDIRECT / UNVERIFIED
- Action: [repoint to X / remove / update slug]

## 4. Hub gaps (P2)
### HUB-01
- Post: [URL]
- Missing: link up to pillar [URL] / links down to [N] supporting posts
- Suggested targets: [URL], [URL], [URL]

## 5. New link recommendations (P3–P4)
### REC-01 | Priority: P3 | Proximity: STRONG
- From: [source URL]
- To: [target URL]
- Insertion point: "...[10–20 quoted words from source]..."
- Anchor text: "[2–6 words]"
- Placement: [wrap existing phrase / NEEDS_NEW_SENTENCE]
- Draft sentence (if needed): [one sentence]
- Reason: [one line]

## 6. Anchor rewrites (P4)
| Source | Current anchor | Target | Suggested anchor |
|--------|----------------|--------|------------------|

## 7. Dropped for density
| Source | Target | Reason |
|--------|--------|--------|

## 8. Open questions
- [item requiring client/owner input]
```

## Failure modes

1. **Recommendations that cannot be placed.** You suggest a link that has no natural home, and the writer skips it. Check: every REC item must contain either a quoted insertion sentence or a drafted sentence. Reject any item with neither.
2. **Link stuffing a single post.** A popular pillar gets 20 new outbound links and dilutes every one. Check: recount existing + recommended links per source post against the density rule before shipping; the "Dropped for density" table should be non-empty on any large audit.
3. **Phantom targets.** You recommend linking to a post that was never in the inventory, or to a slug you misremembered. Check: every target URL in sections 3–6 must appear verbatim in the section 2 inventory table.
4. **Status inflation.** You mark links `OK` when you had no way to verify the URL resolves. Check: if no HTTP check was performed, every non-inventory target is `UNVERIFIED`, and the summary counts must reconcile with the per-item statuses.

## License

MIT
