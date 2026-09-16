---
name: css-refactor-plan
owner: launifycorp
category: Code review
description: This skill turns a legacy stylesheet base into a prioritized, sequenced refactor plan toward a tokendriven design system. You own the deliverable: a ranked list of refactor moves, each with scope, ris...
version: v1
license: MIT
updated: 2026-09-16
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/css-refactor-plan
raw: https://emdly.com/raw/launifycorp/css-refactor-plan.md
install: npx @emdly/cli add launifycorp/css-refactor-plan
---

# CSS Refactor Plan

This skill turns a legacy stylesheet base into a prioritized, sequenced refactor plan toward a token-driven design system. You own the deliverable: a ranked list of refactor moves, each with scope, risk, blast radius, and a verification step, ordered so that the codebase stays shippable after every item. You do not write the final CSS; you decide what gets replaced, in what order, and how the team proves nothing broke.

## When to use

- A re-design is starting on top of a stylesheet that grew by accretion (multiple frameworks, overrides, `!important` chains, dead selectors).
- Design has delivered new tokens, a component inventory, or a Figma library, and engineering needs a migration path from the current CSS.
- Specificity wars are slowing feature work: new components require override hacks to render correctly.
- A team is adopting utility classes, CSS modules, or a component library and must decide what to keep, wrap, or delete.
- Bundle size or style duplication has become a measurable regression and someone asked "where do we start."

Do not use when:

- The task is a single-component style fix or a visual bug with a known cause — go straight to the fix.
- No production CSS exists yet (greenfield). Write a system spec instead; there is nothing to refactor.

## Inputs

Collect before starting:

- **Stylesheet surface**: file tree of CSS/SCSS/LESS/CSS-in-JS, total size, build config, and how styles are loaded (global, module, runtime).
- **Component inventory**: list of UI components with the pages they appear on, or a route list if no inventory exists.
- **Target design system**: token set (color, spacing, typography, radius, elevation, motion), or the Figma file that defines them.
- **Constraints**: browser support matrix, framework version, whether a visual regression suite exists, freeze windows, team size and sprint length.
- **Risk map**: which routes are revenue-critical or legally sensitive (checkout, auth, accessibility-audited pages).

If missing, ask for them in this order and stop until you get the first two:

1. "Point me at the stylesheet entry points and the build config." (blocking)
2. "Which routes or components are highest traffic or highest risk?" (blocking)
3. "Do the new tokens exist yet, and in what format?" (if absent, plan assumes you will derive tokens from current usage in Step 3 and mark them `PROVISIONAL`)
4. "Is there a visual regression or screenshot test suite?" (if absent, every item in the plan gets a manual verification step and risk increases one level)

Never invent token values or component boundaries. Mark them `ASSUMED` and list them in the Open Questions section of the output.

## Method

1. **Measure before judging.** Compute total stylesheet bytes, number of selectors, max specificity score, `!important` count, and duplicate declaration count. Use a static analyzer if available; otherwise grep and count. If you cannot measure, say so and downgrade every priority claim to qualitative. Record baseline numbers — they are the plan's success criteria.

2. **Map selectors to surfaces.** For each stylesheet or block, determine which routes and components it affects. Rule: if a selector matches nothing in the current markup after a full-route crawl or a coverage run, classify it `DEAD` and route it to the deletion batch. If matching is uncertain, classify `UNKNOWN` — never `DEAD` by guess.

3. **Extract the de facto token set.** Cluster every literal color, spacing, font-size, radius, and duration value by frequency. Rule: a value used 5+ times is a token candidate; a value used 1–4 times is drift and must map to the nearest candidate within tolerance (colors ΔE < 3, spacing within one step of the target scale). Values outside tolerance are flagged as intentional exceptions requiring a design decision, not auto-mapped.

4. **Classify every component into one of four dispositions.** Use this rule set, applied in order: `DELETE` if unreferenced; `ADOPT` if it already matches the target system with only token swaps needed; `REWRITE` if its DOM structure or specificity prevents token swapping; `WRAP` if it is third-party or too risky to touch — isolate it behind a boundary (cascade layer, shadow DOM, scoped container) instead. Every component gets exactly one disposition.

5. **Score priority.** For each item compute: `impact` (surfaces touched × traffic weight, 1–5), `effort` (engineer-days, 1–5), `risk` (1–5, +1 if no visual regression coverage, +1 if on a critical route). Priority score = `impact / (effort × risk)`. Rank descending. Break ties by preferring the item that unblocks the most other items.

6. **Sequence with dependency rules.** Enforce this order regardless of score: (a) establish the cascade foundation — layers or a reset — before any component work; (b) land tokens before any component that consumes them; (c) delete dead CSS only after coverage evidence exists; (d) never schedule two `REWRITE` items touching the same DOM subtree in the same phase. Reorder scores only to satisfy these rules, and note each reorder.

7. **Cut phases by shippability.** Group the sequence into phases where each phase ends with a deployable, visually-unchanged (or intentionally-changed) state. Rule: a phase is too large if it exceeds two sprints or touches more than 30% of the stylesheet. Split it.

8. **Attach a verification step to every item.** Choose from: screenshot diff on named routes, computed-style assertion, bundle-size delta threshold, axe/contrast check, or documented manual QA script. Rule: any item on a critical route requires at least two verification methods, one automated.

9. **Define the stop-the-bleeding guardrails.** Specify what prevents regression during the migration: lint rules (no raw hex, no `!important`, specificity ceiling), a CI budget on stylesheet bytes, and a rule for where new CSS goes. Guardrails ship in Phase 0.

10. **Write the plan and state what you did not cover.** Every `UNKNOWN`, `ASSUMED`, or `PROVISIONAL` item from earlier steps goes into Open Questions with the specific person or artifact needed to resolve it.

## Rules

- Never delete CSS based on inspection alone. Deletion requires coverage data, a route crawl, or a git history argument that the feature was removed. Absent evidence, the item is `UNKNOWN` and goes to a research task.
- Never propose a big-bang rewrite. If the plan has no shippable intermediate state, it is wrong — re-cut phases.
- Never invent token values. Derive them from measured usage or take them from the supplied design source. Anything else is `PROVISIONAL` and blocked on design sign-off.
- Never bump specificity to win a conflict. Use cascade layers, scoping, or removing the competing rule. If a specificity increase is genuinely unavoidable, document why in the item and cap it at one level.
- Respect the browser support matrix. Do not propose cascade layers, `:has()`, container queries, or nesting without confirming support; if support is unclear, include a fallback strategy in the item.
- Cap the plan at 25 line items. If you exceed that, batch similar work into single items rather than shipping an unreadable list.
- Effort estimates are in engineer-days and must be ranges, not points. If you cannot estimate, write `NEEDS SIZING` rather than guessing.
- Do not restyle and restructure in the same item. Separate DOM changes from style changes so diffs stay reviewable.
- Accessibility regressions are blocking, not backlog items. Any item that changes color, focus, or motion carries a contrast/focus-visibility check.
- If the input set is incomplete, produce the plan for what you can verify and mark the rest `BLOCKED — needs <input>`. Do not fill gaps with plausible-sounding content.

## Output format

````markdown
# CSS Refactor Plan: <project>

**Date:** <YYYY-MM-DD>
**Scope:** <files / packages covered>
**Excluded:** <what is out of scope and why>

## Baseline metrics

| Metric | Current | Target | How measured |
|---|---|---|---|
| Stylesheet bytes (gzip) | | | |
| Selector count | | | |
| Max specificity | | | |
| `!important` count | | | |
| Distinct color values | | | |
| Distinct spacing values | | | |
| Unused CSS (est. %) | | | |

## Target architecture

- **Cascade strategy:** <layers / scoping / modules — and the layer order>
- **Token layer:** <format, location, naming convention>
- **Component layer:** <where component styles live, boundary rules>
- **Escape hatch:** <what is allowed for one-offs and how it is reviewed>

## Token extraction summary

| Category | Current distinct values | Proposed tokens | Drift to remap | Exceptions needing decision |
|---|---|---|---|---|
| Color | | | | |
| Spacing | | | | |
| Typography | | | | |
| Radius | | | | |
| Elevation | | | | |
| Motion | | | | |

## Component dispositions

| Component | Surfaces | Disposition | Reason |
|---|---|---|---|
| <name> | <routes> | ADOPT/REWRITE/WRAP/DELETE | <one line> |

## Prioritized plan

### Phase 0 — Guardrails (ship first)
| # | Item | Impact | Effort (d) | Risk | Score | Verification |
|---|---|---|---|---|---|---|
| 0.1 | <lint rules, CI budget, new-CSS policy> | | | | | |

**Ships as:** <deployable state at end of phase>

### Phase 1 — <name>
| # | Item | Impact | Effort (d) | Risk | Score | Depends on | Verification |
|---|---|---|---|---|---|---|---|
| 1.1 | | | | | | | |

**Ships as:** <deployable state>
**Rollback:** <how to revert this phase>

### Phase 2 — <name>
<same table shape>

### Phase 3 — <name>
<same table shape>

## Deferred / not doing
| Item | Why deferred | Revisit when |
|---|---|---|

## Sequencing constraints applied
- <each reorder made against raw score, with the rule that forced it>

## Open questions
| # | Question | Blocks | Owner needed |
|---|---|---|---|
| Q1 | | | |

## Assumptions
- <every ASSUMED / PROVISIONAL / UNKNOWN item, explicitly labeled>
````

## Failure modes

- **Phantom deletions.** The plan lists dead CSS that is actually used by a rarely-hit state (error pages, print, email templates, admin routes, `prefers-reduced-motion` branches). Check: before any item enters the deletion batch, confirm coverage was collected while exercising error states, print view, authenticated routes, and every media/feature query branch — not just the happy path. If any branch was not exercised, downgrade to `UNKNOWN`.

- **Token set that matches nothing.** Tokens are copied from the design file while the codebase keeps its old values, so the plan produces two parallel systems instead of one. Check: for every proposed token, confirm at least one concrete remap target in current CSS, or label it `NEW — no current usage`. If more than 20% of tokens have no remap target, the design source and codebase are out of sync — escalate before sequencing.

- **Unshippable middle.** Phases depend on each other so tightly that the codebase is visually broken between phase 2 and phase 5. Check: read each phase's "Ships as" line in isolation and ask whether that state could go to production on a Friday. Any phase that cannot is mis-cut — split it or add a compatibility shim item.

- **Priority scores that hide the risk.** High-impact items on critical routes score well because effort looks small, and the plan front-loads exactly the work most likely to cause an incident. Check: sort the plan by risk instead of score and confirm no risk-5 item lands in Phase 1 without two verification methods and a named rollback. If it does, move it later or add coverage as a prerequisite item.

## License

MIT
