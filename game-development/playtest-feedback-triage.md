---
name: playtest-feedback-triage
owner: pixelforge
category: Game development
description: Sorts raw playtest notes into balance, UX, bugs and feel — ranked by how many testers hit it, with repro steps pulled from context.
version: v3
license: MIT
updated: 2026-08-13
recommended: true
security_checked: true
url: https://emdly.com/skills/pixelforge/playtest-feedback-triage
raw: https://emdly.com/raw/pixelforge/playtest-feedback-triage.md
install: npx @emdly/cli add pixelforge/playtest-feedback-triage
---

# Playtest feedback triage

Twenty testers, two hundred notes, one build. This skill turns the pile into a ranked list the team can act on Monday.

## When to use
- After every playtest session, on the raw notes (forms, Discord threads, transcripts, survey answers).
- Before a milestone review, over several sessions.

## Categories
- **Bug** — the game did something it should not (crash, stuck, wrong number). Needs a repro.
- **Balance** — a number is off: too easy, too strong, too slow to earn.
- **UX** — the player could not find, understand or reach something the game offers.
- **Feel** — it works and is understood but is not enjoyable: floaty, unrewarding, tedious. Keep the tester's words; feel is subjective and paraphrase loses it.
- **Praise** — what to keep. Ranked too; the team should know what not to break.

## Process
1. Split every note into single observations.
2. Merge duplicates across testers (same thing, different words). Count testers, not mentions — one tester saying it five times is one.
3. Classify. When an observation fits two categories, choose by the fix: if changing a number fixes it, Balance; if changing a screen fixes it, UX.
4. For bugs, extract repro steps from context: what they were doing, what happened, what they expected. Mark steps you inferred.
5. Rank within category by tester count; break ties by severity.

## Rules
- Quote the tester for Feel items and for anything ambiguous.
- Do not propose design changes. Report the observation and the count; the designer decides.
- Mark inferred repro steps clearly: "(inferred)".
- A note with no observable content ("meh") is dropped and counted in "unusable notes".

## Output format
```
## Session 12 — 18 testers, 214 notes, 9 unusable

### Bugs (4)
1. **[7 testers]** Falling through the bridge in Level 3 after dash. Repro: dash into the bridge from the left ledge (inferred: during the fade-in). Expected: land. Severity: blocks progress.

### Balance (6)
1. **[11 testers]** Boss 2's second phase — "I ran out of arrows every time" (T04, T09, T15).

### Feel (3)
1. **[6 testers]** "Jump feels like the character is underwater" (T02).

### Keep (3)
1. **[13 testers]** The grapple.
```

## License
MIT
