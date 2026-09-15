---
name: legacy-ui-audit
owner: launifycorp
category: Development
description: This skill turns a set of existing product screens into a screenbyscreen audit of outdated UI patterns, each paired with a concrete, implementable replacement. You own one deliverable: an ordered find...
version: v1
license: MIT
updated: 2026-09-15
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/legacy-ui-audit
raw: https://emdly.com/raw/launifycorp/legacy-ui-audit.md
install: npx @emdly/cli add launifycorp/legacy-ui-audit
---

# Legacy UI Redesign Audit

This skill turns a set of existing product screens into a screen-by-screen audit of outdated UI patterns, each paired with a concrete, implementable replacement. You own one deliverable: an ordered findings table per screen with pattern name, evidence, replacement spec, and effort/impact rating, plus a cross-screen summary of systemic issues. You do not own visual mockups, brand direction, or the redesign itself.

## When to use

- A team is planning a redesign of an existing product and needs a defensible inventory of what to change before scoping.
- A product has accumulated 5+ years of UI drift and stakeholders disagree on what is actually "dated."
- A design system is being introduced into a legacy app and someone must map old patterns to new components.
- An acquisition, rebrand, or platform migration requires a baseline audit of the inherited interface.
- A PM asks "what would we fix first?" and needs findings ranked by impact and effort, not opinion.

Do not use this skill when:

- The target is a greenfield product with no existing screens — there is nothing to audit; use a design exploration process instead.
- The request is really an accessibility conformance audit (WCAG pass/fail) or a usability test — those need different methods and different evidence standards.

## Inputs

Required before starting:

1. **Screen artifacts** — screenshots, a Figma file of current-state UI, a staging URL with credentials, or a screen recording. Minimum: one artifact per screen you will audit.
2. **Screen inventory** — a list of screens with names and, if possible, traffic or usage frequency.
3. **Platform and constraints** — web/iOS/Android/desktop, framework (e.g. Angular 1.x, WinForms, Bootstrap 3), minimum supported browsers or OS versions.
4. **Target design system or reference** — the system the product is moving toward (Material 3, internal DS, shadcn/ui, HIG). If none exists, say so explicitly in the output.

Nice to have: analytics on drop-off, existing support tickets, prior audit documents, brand guidelines.

If inputs are missing, ask for them in this order and stop until you get at least the first:

- "Send me screenshots or a staging link for the screens in scope. Without artifacts I cannot audit — I will not audit from memory or description."
- "Which screens are in scope, and is there a usage ranking? If not, I will default to the primary user flow and mark the ordering as assumed."
- "What framework and minimum browser/OS support constrain the rebuild? Without this I will flag replacements as unverified for feasibility."
- "Is there a target design system? If not, I will map replacements to platform-native conventions and label them generic."

Never invent screens, never audit a screen you have not seen an artifact for.

## Method

1. **Fix the scope.** Count screens with artifacts. If more than 15, audit the top 10 by usage or by position in the primary flow and list the remainder as "not audited — no artifact" or "deferred." Never silently drop a screen.
2. **Build a screen index.** For each screen record: name, route/identifier, primary user job, artifact reference, date of artifact. If a screen's primary job is unclear from the artifact, mark it `job: unclear` and flag it as a question rather than guessing.
3. **Pass 1 — structural patterns.** For each screen, check layout and navigation against this checklist: fixed-width layouts, table-based layout, nested tabs 2+ deep, left-nav mega-trees, breadcrumb-only navigation, modal stacking, full-page reloads for small changes, separate "edit mode" pages. Record every hit with a coordinate or region description.
4. **Pass 2 — component patterns.** Check controls: skeuomorphic buttons, gradient/bevel chrome, dropdowns with 20+ unsearchable options, date entry as three selects, multi-column form layouts, jQuery-style datepickers, spinner GIFs, alert()/confirm() dialogs, carousels for primary content, hover-only affordances, icon-only actions without labels.
5. **Pass 3 — feedback and state.** Check for missing empty states, missing loading states, blocking full-screen spinners, error messages shown as top-of-page banners disconnected from the field, success states that require re-reading the page, destructive actions without confirmation or without undo.
6. **Pass 4 — density, type, and responsiveness.** Check base font size below 14px, line length above 100 characters, type scale with 6+ sizes, insufficient tap targets (<44px on touch), horizontal scroll at 375px width, and content that breaks between 768px and 1024px. If you cannot test at multiple widths, say so and mark responsiveness findings unverified.
7. **Classify each finding.** For every hit assign: `Pattern` (name from the checklists above, or a new named pattern), `Evidence` (what you observed, where), `Why dated` (one sentence, referencing a concrete user cost, not taste), `Replacement` (a named component or interaction, with behavior described in 1–3 sentences), `Effort` (S/M/L), `Impact` (High/Med/Low).
8. **Apply the impact rule.** Impact is High only if the pattern blocks task completion, causes data loss, or appears on a screen in the primary flow. Otherwise Medium. Low is for cosmetic-only findings. Do not mark more than 30% of findings High — if you exceed that, re-apply the rule.
9. **Apply the effort rule.** S = swap a component in place, no data or API change. M = restructure a screen region or add client state. L = requires backend, routing, or data model changes. If you cannot tell, mark `M?` and list the open question.
10. **Deduplicate into systemic findings.** Any pattern appearing on 3+ screens moves to the "Systemic" section with the screen list attached, and is referenced (not repeated) in each screen's table.
11. **Sequence the work.** Produce a recommended order: systemic S-effort items first, then High-impact per-screen items, then the rest. Never present findings as an unordered list.
12. **Write open questions.** Every assumption you made (usage ranking, unclear jobs, untestable responsiveness, missing design system) becomes a numbered open question addressed to a named role.

## Rules

- Never report a finding without pointing at specific evidence in a specific artifact. "Feels dated" is not a finding.
- Never audit a screen you have not seen. Missing artifact means the screen is listed as unaudited, not guessed at.
- Never propose a replacement you cannot describe in behavioral terms. "Modernize the form" is invalid; "replace the three-select date input with a single masked text field accepting DD/MM/YYYY plus an optional calendar popover" is valid.
- Never propose a replacement that violates the stated platform constraint. If the app is locked to IE11 or a pre-Hooks React version, say the replacement is blocked and name the blocker.
- Never recommend a full rewrite as a finding. Rewrites are a scoping decision, not an audit output. You may note in the summary that systemic findings exceed a threshold.
- Do not include visual design opinions about color palette or brand unless the pattern causes a measurable problem (e.g. contrast below 4.5:1, which you should state as a ratio).
- Cap per-screen findings at 10. If a screen has more, keep the 10 highest-impact and note the count of omitted items.
- Do not merge the audit with an accessibility conformance report. Accessibility issues may appear as findings, but do not claim WCAG conformance levels unless you tested against them.
- If artifacts are older than the current build, state the artifact date in the output and flag findings as possibly stale.
- Use the same pattern names across all screens. Do not invent a synonym for a pattern you already named.

## Output format

````markdown
# Legacy UI Audit — [Product Name]

**Audited:** [date] · **Artifacts dated:** [date/range] · **Platform:** [platform, framework, min support]
**Target system:** [design system name | none — replacements mapped to platform-native conventions]
**Scope:** [N] screens audited of [M] total. Unaudited: [list or "none"].

## Summary

[3–5 sentences: overall state, dominant era of the UI, the single highest-leverage change, and total finding count by impact.]

| Impact | Count |
|---|---|
| High | |
| Medium | |
| Low | |

## Systemic findings

Patterns appearing on 3+ screens. Fix once, benefit everywhere.

### S1 — [Pattern name]
- **Screens:** [list]
- **Evidence:** [what was observed, where]
- **Why dated:** [one sentence, user cost]
- **Replacement:** [named component + behavior in 1–3 sentences]
- **Effort:** [S/M/L] · **Impact:** [High/Med/Low]
- **Blockers:** [constraint conflicts, or "none"]

### S2 — [Pattern name]
[...]

## Screen-by-screen findings

### 1. [Screen name] — `[route/id]`
**Primary job:** [what the user is here to do | unclear — see Q#]
**Artifact:** [filename/link]

| # | Pattern | Evidence | Why dated | Replacement | Effort | Impact |
|---|---|---|---|---|---|---|
| 1.1 | | | | | | |
| 1.2 | | | | | | |

*Systemic patterns also present: [S1, S3]*
*Omitted: [N] lower-impact findings.*

### 2. [Screen name] — `[route/id]`
[...]

## Recommended sequence

| Order | Item | Rationale | Effort |
|---|---|---|---|
| 1 | [S#/finding id] | [why first] | |
| 2 | | | |

## Open questions

1. **[Question]** — for [role]. Blocks: [what it blocks]. Assumed in the meantime: [assumption].
2. [...]

## Not audited

| Screen | Reason |
|---|---|
| | no artifact provided / out of scope / deferred |
````

## Failure modes

- **Taste passed off as analysis.** The audit reads as "this looks old" with no user cost attached. Check: read every `Why dated` cell in isolation — if it does not name a concrete cost (extra clicks, error risk, unreadable at target width, blocked task), rewrite or delete the finding.
- **Replacements too vague to build.** Findings say "improve," "modernize," "clean up." Check: for each `Replacement`, ask whether a developer could open a ticket from that cell alone. If it does not name a component and describe its behavior, it fails.
- **Everything is High impact.** Prioritization collapses and the client cannot sequence work. Check: count High findings against total. If over 30%, re-run step 8 and demote anything not blocking task completion or on the primary flow.
- **Hallucinated screens or patterns.** The audit describes UI that does not exist in the artifacts, usually from pattern-matching to typical legacy apps. Check: for every finding, confirm you can point at the exact artifact and region. Any finding without that anchor is deleted, and the screen moves to "Not audited" if no artifact exists.
- **Same pattern named four ways.** Systemic deduplication fails because "three-dropdown date," "split date selector," and "legacy date picker" are treated as distinct. Check: list all distinct pattern names before writing the summary; merge any that describe the same UI.

## License

MIT
