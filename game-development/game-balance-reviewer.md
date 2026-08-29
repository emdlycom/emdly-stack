---
name: game-balance-reviewer
owner: pixelforge
category: Game development
description: Reads stat tables and patch notes, then flags outliers — dominant strategies, dead perks, and curves that punish new players.
version: v2
license: MIT
updated: 2026-08-16
recommended: false
security_checked: true
url: https://emdly.com/skills/pixelforge/game-balance-reviewer
raw: https://emdly.com/raw/pixelforge/game-balance-reviewer.md
install: npx @emdly/cli add pixelforge/game-balance-reviewer
---

# Game balance reviewer

Numbers hide in tables; problems hide in ratios. This skill reads the tables the way a min-maxer does and reports what they would find.

## When to use
- Before a balance patch ships, on the proposed table diff.
- On telemetry (pick rates, win rates, time-to-clear) after a patch.

## Input
Stat tables (units, weapons, perks, economy) as CSV/JSON, the patch notes for the change under review, and — if available — pick/win rates per option.

## Passes
1. **Expected value.** For each option in a group (weapons, perks), compute the comparable number: damage per second, gold per minute, effective HP. Options within ±10% are fine; anything ≥ 25% above the group median is a candidate dominant choice, anything ≥ 25% below is a candidate dead option.
2. **Telemetry cross-check.** If pick rates exist: an option picked by ≥ 40% of players *and* winning ≥ 55% is dominant regardless of the table. An option under 3% pick rate is dead regardless of its numbers — players have already decided.
3. **Early curve.** Compute what a new player has at minutes 0, 5 and 15 against what the content demands at those points. A required stat growing faster than the available stat is a wall; say where.
4. **Patch note honesty.** Do the notes say what the diff does? A "slight adjustment" that is a 30% change gets flagged.

## Rules
- Report ratios and the formula you used, so the designer can disagree with the formula rather than the conclusion.
- Do not propose new values. Flag, quantify, and describe the direction ("bring within 15% of the group").
- A single telemetry week is a hint, not a verdict — say the sample size.

## Output format
```
## Weapons — EV pass (DPS at level 10)
| option | DPS | vs median | pick % | win % | verdict |
| Longbow | 148 | +41% | 46% | 58% | dominant |
| Sling | 62 | −41% | 2% | — | dead |

## Early curve
Minute 5: content requires 220 effective HP; a new player has 160 with any starting perk. Wall at the first miniboss.

## Patch notes
"Slight tuning to Longbow" — the diff is −30% damage. Say so.
```

## License
MIT
