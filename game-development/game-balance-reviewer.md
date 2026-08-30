---
name: game-balance-reviewer
owner: pixelforge
category: Game development
description: Reads stat tables and patch notes, then flags outliers — dominant strategies, dead perks, and curves that punish new players.
version: v4
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/pixelforge/game-balance-reviewer
raw: https://emdly.com/raw/pixelforge/game-balance-reviewer.md
install: npx @emdly/cli add pixelforge/game-balance-reviewer
---

# Game balance reviewer

Numbers hide in tables; problems hide in ratios. This skill reads the tables the way a min-maxer does and reports what they would find, in a form a designer can argue with. Every verdict carries the number it came from, the threshold it was measured against, and the sample it stands on. A designer who disagrees should be able to disagree with the formula, not just the conclusion.

## When to use
- Before a balance patch ships, on the proposed table diff.
- On telemetry (pick rates, win rates, time-to-clear) in the weeks after a patch.
- When a forum or Discord consensus says an option is broken and you want to know whether the data agrees.
- On a new roster addition, to see where it lands against the existing spread.

## Input
Required:
- **Stat tables** as CSV or JSON, with one row per option and a column set that is comparable inside a group (weapons with weapons, perks with perks). Each group must be named.
- **The comparison stat per group**, stated by you or derivable from the columns: DPS, effective HP, gold per minute, clear time.

Optional, and each changes what the skill can say:
- **Patch notes** for the change under review. Without them, pass 4 is skipped, not guessed.
- **Telemetry**: per-option pick count, win count, match count, and the total matches in the sample. Percentages alone are not enough; the counts drive the sample-size arithmetic.
- **Content requirements by time**: what the game asks of a player at fixed minutes.

State the roster size N per group. Every pick-rate threshold below is a function of it.

## Passes

### 1. Expected value, within a group
Compute the comparable number for every option in the group. Take the group median M and the median absolute deviation from it (MAD = median of |option − M|).

- Within ±10% of M: normal spread, no verdict. [judgment, anchored on the fact that most tuning passes move values by less than this, so a smaller gap is inside the noise a designer already accepts]
- 10% to 25% off M: **watch**. Report the number, do not call it dominant or dead.
- 25% or more above M: candidate **dominant**. 25% or more below: candidate **dead**. [judgment, house default]

The 25% rule is genre-blind on its own, so bound it with the group's own spread. If MAD/M is above 0.25, the roster is deliberately spread (a game with a glass-cannon tier) and a flat 25% fires on almost everything. In that case switch the flag threshold to 2 × MAD and say in the output that you did. Report MAD/M for every group either way.

### 2. Telemetry cross-check
Pick rate is meaningless as an absolute number. A 40% pick rate on a 6-option roster is 2.4× what uniform play would give; on a 60-option roster it is 24×. Measure against uniform.

- Uniform share `u = 1/N`. Pick multiple `m = pick share / u`. Report m to two decimals.
- `m ≥ 2.5` → over-picked, candidate dominant. `m ≤ 0.33` → under-picked, candidate dead. Between: no pick-rate verdict. [judgment, house defaults, anchored on the flat 40% / 3% figures this skill used to assert: at N = 6 the multiples give 41.7% and 5.5%, which reproduces the old 40% dominance line at the small roster it was tacitly written for and sets a dead line slightly more generous than the old 3%. At N = 60 the same multiples are 4.2% and 0.55%, which a flat 40% / 3% rule would never catch in either direction]

Win rate stays absolute, because it is a two-sided comparison against a fixed reference, not a share of a roster. In a symmetric matchup the expectation is 50% by arithmetic, not by convention. Flag an option whose win rate sits 5 or more points off that. [judgment on the 5-point band; see the floor below, which is derived from it] If the mode is asymmetric (attacker/defender, first-player advantage), the reference is that mode's own measured baseline, not 50. Say which reference you used.

An option is called **dominant** only when both fire: over-picked *and* win rate above the reference band. Over-picked alone is a popularity finding. High win rate alone on a rarely-picked option is usually a skill-of-the-picker effect; report it as such.

### 3. Sample size, before any verdict
The 95% margin on a win rate is `1.96 × √(p(1−p)/n)`, which at p near 0.5 is `0.98/√n` (normal approximation to the binomial; usable when both n·p and n·(1−p) exceed about 5).

| matches n | 95% margin |
|---|---|
| 30 | ±17.9 pts |
| 100 | ±9.8 pts |
| 385 | ±5.0 pts |
| 1,000 | ±3.1 pts |
| 5,000 | ±1.4 pts |
| 10,000 | ±1.0 pts |

So 385 matches is the floor for a 5-point win-rate claim: `n ≥ (0.98 / band)²` gives 384.2 for a 0.05 band. Change the band and the floor moves with it — a 3-point band needs 1,068 matches. Below the floor, report the margin and write `too small to judge`, unless the deviation itself exceeds the margin, in which case the call stands and you say by how much.

Do not call a dominant strategy off 30 matches. At n = 30 a measured 58% has a true range of 40% to 76%.

For pick shares, the margin is `1.96 × √(p(1−p)/n_total)` on the total matches in the sample. Before calling any option dead, require the expected uniform count `n_total / N ≥ 100`; below that the roster is too sparsely sampled to distinguish a dead option from an unplayed one.

> Thresholds above are defaults; report the thresholds you used.

### 4. Early curve
Sample at fixed wall-clock minutes (0, 5, 15 by default). At each minute compute:
- **Demand**: the largest burst the content can apply before the player can act again, as hits × damage.
- **Available**: base stat + best perk reachable by that minute + expected level-ups by that minute.

Report the margin as an absolute and a percentage of demand. A negative margin is a wall; name the encounter. A demand curve rising faster than the available curve is a wall even where every individual minute passes; say where the lines cross.

### 5. Patch note honesty
Compare each note to its diff. Compute the actual percentage change. A qualitative word attached to a change of 15% or more gets flagged with the real figure. [judgment, house default] Silent changes — a diff with no note — are flagged the same way.

## Rules
- Report the ratio, the formula and the sample for every verdict. A verdict without all three is not shippable output.
- The `### Method` block in the output is mandatory. It lists every threshold and formula actually applied, including any you overrode.
- Do not propose new values. Flag, quantify, and describe the direction: "bring inside the flag band you reported", naming that band's number.
- Never compare across groups. If a table mixes a healer and a damage dealer under one heading, split the group or refuse it; do not median them.
- **One basis per table.** Every row is pre-patch or every row is post-patch, and the table header says which. Never take a proposed value for one option and a live value for another into the same median — the median, every `vs median` ratio and every verdict computed from them would then describe a build that does not exist. Proposed values belong in pass 5, against the basis the table states.
- Never infer telemetry from table values, or table values from telemetry. Missing input is reported as `not supplied`, never as no problem found.
- A single telemetry week is a hint, not a verdict. State the window and the patch it covers.

## Output format

```
## Weapons — group of 8, median DPS 108, MAD/median 0.102 (flat 25% rule applies)

### EV pass (DPS at level 10, pre-patch — the live build this telemetry covers)
| option     | DPS pre-patch | vs median | picks | pick % | m     | matches | win % | 95% margin | verdict            |
| Longbow    | 148           | +37.0%    | 5,704 | 46.0%  | 3.68× | 5,704   | 58.0% | ±1.3       | dominant           |
| Warhammer  | 121           | +12.0%    | 1,984 | 16.0%  | 1.28× | 1,984   | 51.2% | ±2.2       | watch (EV only)    |
| Shortsword | 112           | +3.7%     | 1,240 | 10.0%  | 0.80× | 1,240   | 49.4% | ±2.8       | fine               |
| Spear      | 108           | 0.0%      | 1,116 | 9.0%   | 0.72× | 1,116   | 50.1% | ±2.9       | fine               |
| Crossbow   | 108           | 0.0%      | 1,054 | 8.5%   | 0.68× | 1,054   | 48.8% | ±3.0       | fine               |
| Axe        | 99            | −8.3%     | 806   | 6.5%   | 0.52× | 806     | 47.9% | ±3.5       | fine               |
| Dagger     | 92            | −14.8%    | 248   | 2.0%   | 0.16× | 248     | 52.0% | ±6.2       | dead by pick; win rate too small to judge |
| Sling      | 62            | −42.6%    | 248   | 2.0%   | 0.16× | 248     | 41.5% | ±6.2       | dead               |

Longbow: 46.0% pick is 3.68× the 12.5% uniform share, threshold 2.50×. Pick-share
margin ±0.88 pts (45.1–46.9%, i.e. 3.61×–3.75×), so the multiple clears the
threshold at the bottom of its range. Win rate 58.0% ±1.3 → 56.7–59.3, entirely
above the 55% band. Both conditions fire: dominant.

Sling: 2.0% pick is 0.16× uniform, threshold 0.33×; margin ±0.25 pts (1.75–2.25%,
0.14×–0.18×). Win rate 41.5% on 248 matches is under the 385-match floor, but the
deviation from 50 is 8.5 points against a ±6.2 margin, so the call stands: the true
rate is 35.3–47.7%, below 50 across the whole interval. Dead on both axes.

Dagger: 2.0% pick, 0.16× uniform — dead by pick rate on the same arithmetic as Sling.
Win rate 52.0% ±6.2 spans 45.8–58.2 and straddles 50. Reported as too small to judge,
not as balanced. Rarely-picked options never accumulate the matches their win rate
needs; this cell will stay unjudgeable until the pick rate moves.

Warhammer: +12.0% sits in the 10–25% watch band. No dominance verdict. Win rate
51.2% ±2.2 spans 49.0–53.4 and includes 50.

Direction, no values proposed: Longbow is 37.0% above median against a 25% flag band;
bringing it inside that band means landing at or under 135 DPS. Sling is 42.6% below.

### Early curve
| minute | demand (EHP)                | available (EHP)                          | margin      |
| 0      | 0                           | 120 (base)                               | +120        |
| 5      | 220 (miniboss, 5 hits × 44) | 160 (120 base + 40 best starting perk)   | −60, −27.3% |
| 15     | 240 (Boss 1, 4 hits × 60)   | 250 (120 + 40 perk + 3 levels × 30)      | +10, +4.2%  |

Wall at minute 5, the first miniboss. Demand runs 0 → 240 over the 15 minutes; available
runs 120 → 250. Demand overtakes available somewhere between minute 0 and minute 5 and
falls back behind by minute 15, so the wall is a spike, not a diverging curve. The
minute-15 pass is a 4.2% margin, inside a single level-up.

### Patch notes
Both "from" values below are the pre-patch DPS in the EV table; the "to" values are
proposed and appear nowhere in that table.
"Slight tuning to Longbow" — the diff is 148 → 104 DPS, a change of −29.7%.
"Slight" is not accurate at that magnitude; state the figure in the notes.
Crossbow 108 → 104 (−3.7%) appears in the diff with no note at all.

### Method
- Group: Weapons, N = 8, uniform share u = 1/8 = 12.5%. One weapon per match in this
  mode, so the picks and matches columns carry the same count; in a loadout game they
  would not, and the win-rate margin uses matches, not picks.
- EV: pre-patch table. Median 108, MAD 11, MAD/median 0.102. Below 0.25, so the flat
  bands apply: ±10% fine, 10–25% watch, ≥25% flag. No override.
- Pick: m = share / u. Dominant ≥ 2.50×, dead ≤ 0.33×. Sample n_total = 12,400;
  expected uniform count 12,400 / 8 = 1,550, above the 100 floor, so dead calls are
  permitted. Pick-share margin 1.96 × √(p(1−p)/12,400).
- Win: reference 50% (symmetric mode, confirmed in input). Band ±5 pts.
  Floor n ≥ (0.98/0.05)² = 385. Margin 0.98/√n.
- Curve: minutes 0/5/15, demand = hits × damage in one burst window.
- Notes: qualitative word flagged at ≥15% actual change.
- Window: telemetry is patch 1.7.2, days 3–17, one region. Single-window sample.
```

If telemetry is absent, the pick, m, matches, win and margin columns read `not supplied`
in every row, the verdict column carries EV verdicts only, and the Method block says
`Telemetry: not supplied — no dominance or dead call made on play data.`

## Edge cases
- **No stat tables.** The method collapses; there is nothing to take a median of. Refuse: "No stat tables supplied. This skill compares options inside a group; supply the group and its comparison stat." Do not review from patch notes alone.
- **Group of fewer than 4 options.** The median is unstable and, at N = 3, the middle option can never be flagged against itself. Report every pairwise ratio instead, no median verdicts, and say why.
- **Group larger than 60 options.** Do not print a full table. Print every option outside a flag band, plus the top and bottom deciles, plus the group median, MAD and N. Say how many rows were omitted and that they were inside the bands.
- **Columns are not comparable.** A group mixing roles (a healer and a damage dealer under "units") has no meaningful median. Split it by role if the input names roles, otherwise refuse that group by name and review the rest.
- **You cannot tell which build the table came from.** Do not guess the basis. Ask which build the rows describe, state the answer in the table header, and read the patch diff in that direction. A table whose basis is unknown supports no verdict.
- **Telemetry supplied as percentages with no counts.** Every sample-size test needs n. Report the multiples and the ratios, mark every margin cell `n not supplied`, and make no dominance or dead call. Ask for match counts.
- **Options gated by unlock, level or purchase.** Pick share is confounded by who can pick at all. Normalize by eligible players if the input gives eligibility; if it does not, report the raw share, mark it `availability-confounded`, and make no dead call. An unowned option is not a rejected one.
- **Telemetry window shorter than a week, or spanning a patch boundary.** Report it and label the sample provisional. A window that crosses a patch is two samples, not one; ask for them split.
- **No patch notes.** Skip pass 5 and write `Patch notes: not supplied — honesty pass skipped.` Do not infer intent from the diff.
- **No content-requirement data.** Skip pass 4 and say so. Do not estimate what content demands from the stat table.
- **Table and telemetry point opposite directions.** See the stop below. This is more often an instrumentation fault than a balance finding.
- **An option with zero recorded picks.** Distinguish `0 picks in n_total = X` from `no recorded activity` (the option is absent from the telemetry file). The first is a finding; the second is missing data.

## Stop and hand back
- **A flagged option is sold for money or gated behind a paid pass.** Pick rate is confounded by who bought it, and a nerf to purchased content is a refund and consumer-law question, not a tuning one. Report the numbers, name the confound, and hand to the designer and whoever owns monetization without stating a dominant or dead verdict.
- **Table and telemetry disagree in direction** (the table says weak and the telemetry says dominant, or the reverse). Do not reconcile them into a finding. Stop, say which two numbers conflict, and ask for the telemetry query and the build the table came from before reporting anything.
- **The flag would drive a mid-season change to a ranked or competitive mode.** Report to the season owner as a candidate, marked as such. This skill does not have the ladder-integrity context that call needs.

## License
MIT
