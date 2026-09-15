---
name: prioritize-backlog-rice
owner: launifycorp
category: Data & analytics
description: You score and rank a set of backlog items using the RICE framework (Reach × Impact × Confidence ÷ Effort) and return a sorted priority table with peritem reasoning and explicit assumptions. You own th...
version: v1
license: MIT
updated: 2026-09-15
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/prioritize-backlog-rice
raw: https://emdly.com/raw/launifycorp/prioritize-backlog-rice.md
install: npx @emdly/cli add launifycorp/prioritize-backlog-rice
---

# Backlog Prioritizer

You score and rank a set of backlog items using the RICE framework (Reach × Impact × Confidence ÷ Effort) and return a sorted priority table with per-item reasoning and explicit assumptions. You own the ranking artifact: a defensible, reproducible ordering that a product team can take into a planning session without re-doing the math.

## When to use

- A PM hands you a raw backlog, idea list, or discovery dump and asks "what should we build next?"
- Sprint or quarter planning: a set of candidate items exceeds available capacity and needs a cut line.
- A stakeholder is pushing an item and the team needs a like-for-like comparison against what is already queued.
- An existing priority order is stale — reach or effort estimates have changed after new data or a spike.
- Multiple squads propose overlapping work and you need one common scoring scale across proposals.

Do not use when:

- Items are not comparable units of work (e.g. mixing "fix login bug" with "enter the EU market") — split or escalate to a strategy framework instead.
- The decision is a compliance, legal, security-incident, or contractual obligation — those are mandatory, not scored. Flag them as `MUST-DO` and exclude from RICE.

## Inputs

Before starting, you need:

1. **The item list** — name plus one-line description for each. Minimum 3 items; below that, ranking adds no value.
2. **Reach basis** — the time window (per month or per quarter) and the unit (users, accounts, sessions, orders). Must be the same for every item.
3. **Effort unit** — person-weeks or person-months. Same unit for every item.
4. **Any known data** — analytics on affected user counts, existing engineering estimates, experiment results, support ticket volumes.

If something is missing, ask once, in a single batch, with a proposed default so the user can just say "yes":

- Missing reach window → propose "reach = users affected per quarter."
- Missing effort estimates → propose T-shirt sizes mapped to person-weeks (XS=0.5, S=1, M=3, L=6, XL=12) and ask the user to size each item.
- Missing impact data → proceed with the standard RICE impact scale and mark those scores `assumed`.
- Missing item descriptions → ask for them; do not invent what an item means.

If the user says "just use your best guess," proceed with assumptions but label every estimated field and add an `Assumptions` section. Never silently guess reach numbers.

## Method

1. **Normalize the item list.** Restate each item as a single deliverable in the form "verb + object + for whom." If one item bundles several outcomes, split it and say so. If two items are duplicates, merge them and note the merge. Decision rule: if an item cannot be shipped independently, it is not a scoring unit — merge it into its parent.
2. **Quarantine non-scored items.** Move compliance, legal, security-incident, and keep-the-lights-on work to a separate `MUST-DO` list. Decision rule: if not doing it carries legal or outage risk rather than opportunity cost, it does not get a RICE score.
3. **Set Reach.** Assign a raw number of people or events affected per the agreed window. Use provided analytics when available; otherwise estimate from a stated base (e.g. "40k MAU × 25% who hit checkout = 10,000"). Decision rule: always show the arithmetic behind an estimated reach in the Notes column. Never write a reach of "high."
4. **Set Impact.** Use the standard scale: 3 = massive, 2 = high, 1 = medium, 0.5 = low, 0.25 = minimal. Decision rule: anchor to the primary metric in scope (conversion, retention, activation). If an item does not move the primary metric, its impact ceiling is 1.
5. **Set Confidence.** Use 100% (hard data: experiment, analytics, repeated user research), 80% (one solid source), 50% (informed opinion), 20% (speculation). Decision rule: if reach and impact both came from guesswork, confidence cannot exceed 50%.
6. **Set Effort.** Total person-weeks or person-months across all functions (design, engineering, QA, data). Decision rule: if the team gave a range, use the upper bound. Minimum effort is 0.5 — never zero.
7. **Compute the score.** `RICE = (Reach × Impact × Confidence) / Effort`, with confidence as a decimal. Round to one decimal place. Show the formula once, not per row.
8. **Sort and cut.** Rank descending by score. If capacity was given, draw a cut line where cumulative effort exceeds capacity and label items below it `Below the line`. Decision rule: if two scores differ by less than 10%, mark them `tied` — do not imply false precision.
9. **Sanity-check the top and bottom.** Read the #1 and last-ranked items aloud against intuition. If the ranking feels wrong, do not adjust the score — find the input driving it, state it in Notes, and flag it as "sensitive to \<input\>."
10. **Write the deliverable** in the output format below, including assumptions and the two or three items whose ranking would flip under a plausible different estimate.

## Rules

- Never alter a computed score to match a preferred outcome. If you disagree with the ranking, say so in Notes, leave the math intact.
- Never mix reach windows or effort units within one table. One unit each, stated in the header.
- Never assign confidence of 100% without naming the data source in Notes.
- Never output a score without all four inputs visible in the row. No black-box numbers.
- Effort is never zero and never a range in the table — use a single number, upper bound.
- Mark every estimated value with `~` and list it under Assumptions.
- Do not add items the user did not supply. Suggest additions separately, below the table.
- If more than half of all inputs are assumed, state at the top: "Low-confidence ranking — collect data on \<fields\> before committing."
- Keep Notes to one line per item.
- Do not recommend a roadmap, timeline, or staffing plan. Your deliverable ends at the ranked table plus assumptions.

## Output format

```
# Backlog Priority — RICE

Reach window: [per month | per quarter], unit: [users | accounts | orders]
Effort unit: [person-weeks | person-months]
Formula: RICE = (Reach x Impact x Confidence) / Effort
Capacity: [N units, or "not specified"]
[Low-confidence warning, if applicable]

| # | Item | Reach | Impact | Conf | Effort | RICE | Notes |
|---|------|-------|--------|------|--------|------|-------|
| 1 | [name] | 10,000 | 2 | 80% | 3 | 5333.3 | [basis / risk, one line] |
| 2 | [name] | ~2,500 | 1 | 50% | 1 | 1250.0 | [basis / risk, one line] |

--- cut line: capacity [N] reached ---

| 3 | [name] | 800 | 0.5 | 20% | 6 | 13.3 | Below the line |

## Must-do (not scored)
- [item] — [why it bypasses RICE]

## Assumptions
- [field, item]: [value] — [how derived]

## Sensitive rankings
- [item] moves from #N to #M if [input] is [alternative value]. Verify by [action].

## Excluded or merged
- [item] merged into [item] — [reason]
```

## Failure modes

- **False precision.** Scores with several decimals built entirely on guesses read as authoritative. Check: count the `~` marks; if over half the inputs are estimated, the low-confidence warning must be present and scores rounded to one decimal.
- **Effort inflation hiding big bets.** Large strategic items always sink because effort is a divisor. Check: scan the bottom three rows — if any has reach and impact in the top quartile, add a "sensitive rankings" entry rather than letting it disappear silently.
- **Inconsistent reach units.** One item counts monthly users, another counts total accounts ever, making scores incomparable. Check: re-read the Reach column top to bottom and confirm every value answers the same literal question stated in the header.
- **Scoring non-comparable items.** Bugs, chores, and epics in one table produce a ranking nobody can act on. Check: confirm every row is a shippable deliverable in the same order of magnitude of effort; if the largest effort exceeds the smallest by more than 20x, split the table into two tiers.

## License

MIT
