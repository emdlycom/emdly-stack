---
name: playtest-feedback-triage
owner: pixelforge
category: Game development
description: Sorts raw playtest notes into balance, UX, bugs and feel — ranked by how many testers hit it, with repro steps pulled from context.
version: v4
license: MIT
updated: 2026-08-30
recommended: true
security_checked: true
url: https://emdly.com/skills/pixelforge/playtest-feedback-triage
raw: https://emdly.com/raw/pixelforge/playtest-feedback-triage.md
install: npx @emdly/cli add pixelforge/playtest-feedback-triage
---

# Playtest feedback triage

Twenty testers, two hundred notes, one build. This skill turns the pile into a ranked list the team can act on Monday. The output is for the person who has to pick what gets fixed this week, so every item carries how many distinct testers hit it, out of how many played, and enough of the tester's own words that the fix is not designed from a paraphrase.

## When to use
- After a playtest session, on the raw notes: forms, Discord threads, session transcripts, survey free-text.
- Before a milestone review, rolled up across several sessions of the same build family.
- When two testers disagree loudly and you need to know whether either is a majority.
- When the team wants to know what to protect, not only what to fix.

## Input
Required:
- **The notes**, as text. Any mix of forms, exports, transcripts and threads.
- **A tester identifier per note** (T01, a handle, a row id). The counting rule and every quote attribution depend on it. Without ids the method changes; see Edge cases.
- **The tester count** for the session, and the build or version tested.

Optional:
- Session length and what testers were asked to do. Used to judge whether "never found it" means hidden or means out of scope.
- Prior sessions' output, for a roll-up.

## Categories
Closed list. Every observation lands in exactly one.

- **Bug** — the game did something it should not: crash, stuck, wrong number, wrong art. Needs a repro or an explicit note that none was obtainable.
- **Balance** — a number is off: too easy, too hard, too strong, too slow to earn.
- **UX** — the tester could not find, understand or reach something the game already offers.
- **Feel** — it works and is understood but is not enjoyable: floaty, unrewarding, tedious. Keep the tester's words; feel is subjective and paraphrase loses it.
- **Praise** — what to keep. Ranked too; the team should know what not to break.
- **Unresolved** — fits two categories and the tiebreak in pass 3 does not settle it. Listed with both candidates named, for the designer to route. Not a dumping ground: if pass 3 settles it, it is settled.

## Passes

1. **Split.** Break every note into single observations. One observation names one object and one outcome. "The jump feels floaty and I fell through the bridge" is two.
2. **Merge.** Two observations merge when they name the same object *and* the same failure. Same object, different failure stays separate: "the bridge is invisible" and "I fell through the bridge" are two items. Count **distinct testers, not mentions** — one tester saying a thing five times across a thread is one. In a roll-up, one tester across three sessions is still one.
3. **Classify.** Choose by the fix, in this order: if changing a number fixes it, Balance; if changing a screen, label or layout fixes it, UX; if changing code behaviour fixes it, Bug; if nothing is broken and nothing is unclear, Feel. If two survive that order, it is Unresolved.
4. **Repro.** For every Bug, extract: what they were doing, what happened, what they expected. Mark any step you supplied from context with `(inferred)`. If context gives you none of the three, write `no repro obtainable` rather than inventing one.
5. **Severity.** Closed list, in rank order: `blocks` (could not continue) · `loses` (progress, save or input lost) · `degrades` (worked, but wrongly) · `cosmetic` (visible, no effect on play).
6. **Rank.** Within each category, by distinct tester count descending. Break ties by severity order above, then by earliest first report. Items with one tester are not ranked; they go under a `Single-report` heading at the end of their category, in the order found.

## Rules
- Report count and share for every item: `[7/18 testers, 38.9%]`. The denominator is the tester count, not the note count.
- Quote the tester verbatim for every Feel item, every Unresolved item, and anything where your classification depended on reading intent. Attribute by id. One quote per item, under 25 words; trim with `…` inside the quote only.
- Each item is one line, under 40 words excluding the quote and the repro block.
- Do not propose design changes. Report the observation, the count and the severity. The designer decides. A tester's own proposed fix is quoted as theirs, never adopted as a recommendation.
- Mark inferred repro steps `(inferred)`. Never mark a step inferred that the tester actually wrote.
- Praise is counted and ranked by the same rules as everything else. Do not soften a low count by moving an item into Praise.
- Report the arithmetic in the header so it reconciles: notes in, unusable, usable, observations after splitting, distinct items after merging.
- Under 5 testers, do not rank. See Edge cases.

## Output format

```
## Session 12 — build 0.9.4-rc2
18 testers · 214 notes · 9 unusable · 205 usable → 331 observations → 118 distinct items
Bug 21 · Balance 34 · UX 29 · Feel 22 · Praise 12 = 118
Every category is printed in full in the delivered file. This excerpt shows the top of
each; the remaining items follow the same shape, in rank order, to the count in brackets.

### Bugs (21)
1. **[7/18, 38.9%]** `blocks` — Falling through the bridge in Level 3 after a dash.
   Repro: stand on the left ledge, dash toward the bridge (inferred: during the fade-in
   after the checkpoint). Expected: land on the planks. Actual: fall to the death plane.
2. **[4/18, 22.2%]** `loses` — Quitting from the pause menu during a boss discards the
   run's collected shards. Repro: no repro obtainable — three testers reported the
   outcome, none recorded what they did before the quit.
3. **[3/18, 16.7%]** `degrades` — Arrow count in the HUD reads 12 while the inventory
   reads 9 after picking up a quiver. Repro: pick up a quiver at full arrows.
   Expected: both read the same. Actual: HUD is high by the pickup amount.

Single-report bugs
- **[1/18, 5.6%]** `cosmetic` — Rain renders in front of the pause overlay (T11).

### Balance (34)
1. **[11/18, 61.1%]** — Boss 2's second phase outlasts the arrow supply.
   "I ran out of arrows every time and just circled him" (T04). Also T09, T15.
2. **[6/18, 33.3%]** — Shard cost of the second grapple upgrade is unreachable in one
   run. Testers reported ending runs at 60–80% of the cost.

### UX (29)
1. **[9/18, 50.0%]** — The map legend does not say which icon is a save point; testers
   used the shrine icon and lost progress twice.
2. **[5/18, 27.8%]** — Nobody found the loadout screen. Entry point is the third tab of
   a menu reached by holding a button that is never prompted.

### Feel (22)
1. **[6/18, 33.3%]** — "Jump feels like the character is underwater" (T02).
2. **[4/18, 22.2%]** — "Landing a hit is silent, so I'm never sure I connected" (T07).

### Praise (12)
1. **[13/18, 72.2%]** — The grapple. "The grapple is the whole reason I kept going" (T05).
2. **[5/18, 27.8%]** — The checkpoint density in Level 2.

### Unresolved (2)
- **[3/18, 16.7%]** — Bug or Balance: the second miniboss "doesn't seem to take damage
  from the spear" (T14). Either the spear's damage type is wrong (Bug) or its number
  against armoured targets is low (Balance). Needs a code answer to route.
- **[2/18, 11.1%]** — UX or Feel: "the menu is fine, I just never wanted to open it"
  (T03). Nothing is unfindable and nothing is broken.

### Unusable notes (9)
Dropped, not counted in any category: 6 with no object named ("meh", "it was ok"),
2 rating-only survey rows with empty free-text, 1 note about a build not under test
(0.9.2). Ids on the dropped notes: T01, T03, T06, T08, T08, T12, T16, T17, T18 — nine
notes from eight testers, T08 twice. Those testers still count once on their usable items.

### Method
Merge rule: same object and same failure. Counting unit: distinct testers, n = 18.
Classification tiebreak: number → screen → code → nothing broken.
Ranked by tester count, ties by severity order blocks > loses > degrades > cosmetic.
Single-report items listed unranked. Notes 214 = 205 usable + 9 unusable;
205 usable produced 331 observations, merged to 118 distinct items.
```

Second shape, when the sample is below the ranking floor:

```
## Milestone roll-up — sessions 13–15, builds 0.9.5 through 0.9.7
4 unique testers · 22 notes · 6 unusable · 16 usable → 24 observations → 16 distinct items
Bug 3 · Balance 5 · UX 6 · Feel 2 · Praise 0 = 16

Not ranked: 4 testers is below the 5-tester floor. At n = 4 one tester is 25% of the
sample, so ordering by count would present noise as priority. Items are listed by
category in the order found, with raw counts.

### Bugs (3)
- **[2/4]** `blocks` — Elevator in the hub does not accept input after a fast-travel.
  Repro: fast-travel to hub, step onto the elevator, press up. Expected: it rises.
  Actual: nothing, until the hub is reloaded.
- **[1/4]** `degrades` — Subtitles keep the previous line on screen through the next one.
  Repro: no repro obtainable — reported from a transcript aside with no context.
- **[1/4]** `cosmetic` — The shard counter overlaps the minimap at ultrawide aspect.
  Repro: set 21:9, open the hub. Expected: no overlap. Actual: counter sits over the
  minimap's right edge (inferred: only in the hub, the only scene the tester reached).

### Praise (0)
None recorded. Not evidence of a problem; across three sessions no tester was asked
what they liked, and the form for 13–15 had no positive prompt.
```

## Edge cases
- **No notes, or every note unusable.** Report the header line with zeros and stop: `0 usable notes of 22 · no items`. Do not produce an empty ranked list that reads like a clean session.
- **A note with no observable content.** "meh", "it was fine", a rating with empty free-text. Dropped, counted in `Unusable notes`, tester id retained so the same tester's usable notes still count once. "Loved it" with no referent is unusable; "loved the grapple" is Praise.
- **Fewer than 5 testers.** Do not rank and do not report shares as percentages; a single tester is 20% or more of the sample and the ordering is noise. List by category in the order found with raw counts, and print the banner shown in the second example. [judgment, house floor — chosen so that one tester can never exceed a fifth of the sample]
- **No tester identifiers.** Quoting and the distinct-tester count both fail. Say so: the skill cannot count testers, so it counts *notes* instead, labels every count `notes, not testers`, drops all attribution, and states in the Method block that the ranking is by mention volume. This is a weaker method; do not present it as the normal output.
- **Notes are a transcript with no speaker labels.** Same as above, plus: do not split by paragraph and assume speaker changes. Ask for a labelled transcript before triaging a session that is transcript-only.
- **One tester submitted twice** (a form and a Discord thread). Merge to one tester by id before counting. If the two submissions conflict, keep both observations and mark the item `same tester, conflicting reports`.
- **Testers agreeing in a thread** ("+1", "same here"). An agreement inherits the item and adds that tester once. A reply that adds a different failure is a new observation, not an agreement.
- **Two testers contradict each other** ("the boss is too easy" / "the boss is impossible"). Do not merge and do not average. File both, each with its count, and put them adjacent so the split is visible.
- **A note that is entirely a design proposal** ("you should add a dodge roll"). It is not an observation. Extract the underlying complaint if one is stated and file that; if none is, file the proposal under its category with the tester's words quoted and marked `proposal, no observation`. Never adopt it.
- **A note about a build the team did not ship, or an older build.** Unusable, counted, and the build named in the unusable line so the discrepancy is visible.
- **Notes in a language you are not triaging in.** Translate the observation, keep the original text in the quote alongside the translation, and mark the item `translated`. Do not drop it.
- **More notes than you can hold at once** (roughly 500+, or several sessions at once). Triage session by session to distinct items, then merge the per-session item lists. Never sample the notes: a bug two testers hit is exactly the kind of item sampling deletes. Say in the header how many sessions were merged and how many testers were unique across them.
- **Session context missing.** Without knowing what testers were asked to do, "never found the loadout screen" may mean it is hidden or may mean it was out of scope. File it as UX and mark it `scope unknown — session brief not supplied`.

## Stop and hand back
- **Any note reporting a physical reaction**: photosensitivity, seizure, migraine, nausea, motion sickness. Route to whoever owns accessibility before triage, with the note verbatim. Do not file it as Feel and do not rank it by tester count; one report is enough.
- **Any note reporting harassment, abuse or a safety incident** in a multiplayer or in-person session. Route to whoever runs the playtest programme. It does not enter the ranked list.
- **A note containing personal data** (real name, address, contact details, employer) or identifying a tester as a minor. Strip it from the triage output, keep the observation, and hand the raw note to whoever owns tester records.
- **A note that names unannounced content** where the roll-up will circulate beyond the team. Flag the item and ask who the distribution list is before writing it up.

## License
MIT
