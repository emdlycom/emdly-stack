---
name: prd-sharpener
owner: deckhand
category: Product
description: Turns a draft PRD into one a team can build from — problem before solution, success metric with a number, explicit non-goals, and open questions with owners.
version: v2
license: MIT
updated: 2026-08-29
recommended: false
security_checked: true
url: https://emdly.com/skills/deckhand/prd-sharpener
raw: https://emdly.com/raw/deckhand/prd-sharpener.md
install: npx @emdly/cli add deckhand/prd-sharpener
---

# PRD sharpener

A PRD is done when an engineer can say what *not* to build. This skill edits toward that.

## When to use
- On any product doc before design or engineering review.
- On a feature request that arrived as a Slack message and needs to become a document.

## Input
The draft (any structure), the metric the team already tracks that this should move, and the constraints (deadline, platforms, dependencies).

## The sharpened structure
1. **Problem** — who, what they are trying to do, what goes wrong today, evidence (a quote, a number, a support ticket count). No solution words in this section.
2. **Success** — one primary metric with a current value and a target, and the date it will be read. Secondary metrics at most two. Guardrail metrics (what must not get worse).
3. **Solution** — the smallest thing that could move the metric, described as behavior, with the two alternatives considered and why not.
4. **Non-goals** — explicit list of what this does not do, especially the things a reader would assume it does.
5. **Scope in stages** if it does not fit one release, with what each stage proves.
6. **Open questions** — each with an owner and a date; a question without an owner is a risk, and is labeled as one.
7. **Risks** — what would make this fail even if built correctly.

## Rules
- Move solution language out of the problem section; quote the moved sentences so the author sees the change.
- A success metric without a current value is flagged; without a target it is rejected. "Increase engagement" is not a metric.
- Sanity-check the metric: can the feature plausibly move it by that amount in that time? Say if not.
- Do not add features. Sharpen what is there; propose cuts, not additions.
- Keep the author's voice and terminology.

## Output format
```
## Problem (rewritten)
Shop owners who import product feeds cannot see which rows were dropped; 34 support tickets in Q2 ask "why is product X missing" (source: Zendesk tag `feed-missing`).
> moved to Solution: "we should add a validation report page"

## Success
Primary: feed-related support tickets/week — 8.5 now → ≤ 3 by 2026-11-30. Guardrail: import time p95 stays under 90 s.

## Non-goals
- Fixing rows automatically · Supporting Shopify-format feeds · Historical reports for imports before launch

## Open questions
| question | owner | by |
| do we keep dropped rows for 30 or 90 days? | data lead | 2026-09-05 |
| — "is legal OK with storing raw feeds?" — **no owner: risk** |
```

## License
MIT
