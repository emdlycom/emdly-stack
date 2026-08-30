---
name: sprint-retro-facilitator
owner: opsmith
category: Ticket ops
description: Prepares a retrospective from the sprint's data — what shipped, what slipped, where time went — and turns it into three concrete questions for the team.
version: v3
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/opsmith/sprint-retro-facilitator
raw: https://emdly.com/raw/opsmith/sprint-retro-facilitator.md
install: npx @emdly/cli add opsmith/sprint-retro-facilitator
---

# Sprint retro facilitator

A retro is only useful when it starts from facts. This skill turns the sprint's tickets and PRs into a short evidence sheet and three questions worth an hour of the team's time. It reports numbers and asks questions. It does not diagnose, does not propose solutions, and never attaches a name to a slow number.

## When to use
- The last day of a sprint, before the retro is scheduled.
- After a release that went badly, over any date range.
- Before a quarterly review, run over the last three sprints to see whether the same carry-overs keep appearing.
- When the team says "the retro is always vibes" — this is what replaces the vibes.

## Input
- **Tickets in the sprint**, each with its transitions and timestamps (state, from, to, when). Transitions are the method; ids and titles alone are not enough.
- **Which tickets were committed at sprint start**, and which were added after. If the board does not record this, say so — see Edge cases.
- **PRs** with open and merge timestamps, and the ticket each maps to if the link exists.
- **The sprint goal**, in the words the team wrote it.
- **The previous retro's action items**, if the team keeps them.
- **The previous sprint's carry-over list**, if you want the *slipped twice* line. Without it, a ticket that has slipped for three sprints looks identical to one slipping for the first time, and the sheet says `slipped twice: not determinable (no previous sprint supplied)`.

> Thresholds above are defaults; report the thresholds you used.

## Process
1. **Count honestly.** Split the sprint into two populations and keep them split: the committed set and the mid-sprint additions. For each, report delivered and carried over. `committed = delivered + carried over` must hold, and so must the addition line; if either does not, the input is inconsistent and you say so instead of publishing a total. Carry-overs that were also carried over last sprint get their own list: *slipped twice*.
2. **Where the time went.** Cycle time per delivered ticket, measured from the first entry into an in-progress state to the first entry into a done state. Boards rename these, so resolve them once, at the start, and print the mapping you used: an in-progress state is any state the team moves work into to begin it (In Progress, In Development, Doing); a done state is any terminal state that counts as shipped (Done, Closed, Released). "In Review", "Blocked" and "QA" are neither — they sit inside cycle time, not at its edges. If the board's states do not map cleanly onto those two groups, say which states you treated as which. Report the median and the 85th percentile, and always report `n` next to them. Time in review (PR opened → merged) is reported separately, as its own median, on the PRs that link to a ticket.
3. **Previous actions.** For each action item from the last retro, one of four verdicts, and only these four:

   | verdict | what earns it |
   |---|---|
   | done | a ticket, PR, commit, or board setting in this sprint's data that the action names |
   | partly | evidence exists for some of the action's scope and not the rest — say which half |
   | not started | the sprint's data contains nothing matching, and the action was checkable from it |
   | not checkable | the action names something outside tickets and PRs ("talk to design more") |

   Never write "done" from an assertion. If the only evidence is that someone said it was done, that is `not checkable`.
4. **Three questions.** Rank the candidate observations by the size of the gap between plan and reality, and take the top three. The candidates come from a closed list: the committed-vs-delivered gap · the mid-sprint addition count · anything on the *slipped twice* list · a stage of cycle time that is more than half the total · a previous action that is `not started` · a stated sprint goal that the delivered tickets do not cover · a data gap large enough to make one of the counts unreportable. Phrase each as one neutral sentence ending in a question mark, naming the number it came from. Never phrase a question so that only one answer fits.

## Rules
- Facts and questions only. Never assign blame, and never name who was slow. Cycle time is a team number: no per-person cycle time, no per-person ticket counts, no "X's tickets took longest". Ticket ids are fine; people's names appear only as an action item's owner.
- Do not propose solutions. That is the team's job in the room, and a sheet that arrives with answers ends the conversation before it starts.
- If the sprint goal is missing, say so as the first observation. That is usually the finding.
- Under 40 lines, matching this owner's `opsmith/standup-synthesizer`, which already says "under 40 lines". [house rule] The evidence block is at most 12 bullets and the three questions are one sentence each.
- Every number in the sheet carries its `n` or its arithmetic in brackets, so anyone in the room can check it. No percentage appears without the two numbers it came from.
- Exactly three questions. Not two, not five. Three is what fits in the hour alongside the discussion, and a longer list gets triaged in the room instead of answered. [house rule] If a fourth observation is genuinely larger than one of your three, swap it in; do not append it.
- The sheet is circulated before the retro, not read aloud in it. Write it to be read cold by someone who was not in the sprint.
- p85 is a convention, not a standard: it is used here because it names the slow tail without letting one outlier set the number, the way a max would. [judgment] Below the sample floor in Edge cases, do not report it.

## Running it over several sprints
When the input covers more than one sprint — a quarterly review, or "why does this keep happening" — do not merge them into one population. Run steps 1 and 2 per sprint and print one row per sprint, so the shape over time is visible: committed, delivered, added, carried, median cycle time, p85, n. Then run steps 3 and 4 once, across the whole range.

Two things change. First, the *slipped twice* list becomes *slipped n times*, and a ticket that appears in three consecutive carry-over lists is its own question regardless of how it ranks. Second, do not average the medians. A median of medians is not a median; if you need a range-wide figure, pool the delivered tickets and compute it once, and say that is what you did.

Cap it at six sprints. Beyond that the row table alone breaks the 40-line rule, and the question is no longer a retro question. [house rule]

## What does not go on the sheet
A closed list. None of these appear, however available the data is:
- Velocity, story points, or points delivered. They are an estimate of an estimate, and putting them next to real durations lends them a precision they do not have.
- Anything per person: cycle time, ticket count, review count, commit count.
- A comparison to another team, or to an industry benchmark.
- A cause. "Review was slow because the reviewers were on the release" is a hypothesis; it belongs in the room, from the people who were there.
- A recommendation, a next step, or an action item. The team writes those at the end of the retro.

## Output format
```
## Sprint 34 — evidence
- Sprint goal: (not stated on the board) — nothing here can be measured against it.
- Committed 21 · delivered 14 · carried over 7
- Added mid-sprint 6 · delivered 4 · carried over 2
- Total delivered 18 · total carried 9 (slipped twice: PROJ-88, PROJ-91)
- Cycle time (n=15 of 18 delivered; 3 have no recorded transitions): median 2.4 d · p85 6.1 d
- Time in review (n=15): median 1.3 d — 54% of median cycle time [1.3 / 2.4]
- Last retro's actions: "pair on flaky tests" — done (7 paired commits on `flaky-*` branches)
- Last retro's actions: "limit WIP to 2" — not started (no evidence found; board has no WIP limit set)
- Last retro's actions: "talk to design earlier" — not checkable (names no ticket or PR)
- PROJ-96, PROJ-101, PROJ-103 delivered with no transitions recorded — excluded from cycle time

## Three questions
1. Six tickets arrived mid-sprint and seven of the committed ones did not finish — what decides what gets in?
2. Review is 54% of median cycle time — what is review waiting on?
3. PROJ-88 has now slipped two sprints running — what is different about it?
```
Ten evidence bullets, three questions, 16 lines of sheet.

Small sprint, thin data:
```
## Sprint 35 — evidence
- Sprint goal: "make the importer resumable" — stated.
- Committed 4 · delivered 3 · carried over 1
- Added mid-sprint 0 · delivered 0 · carried over 0
- Cycle time (n=3): too small to judge. Range 1.1–9.4 d. No median, no p85 reported.
- Time in review: no PRs linked to tickets — not measured.
- Last retro's actions: none recorded.

## Three questions
1. PROJ-140 took 9.4 days against 1.1 and 1.3 for the other two — what happened to it?
2. No PR was linked to a ticket this sprint — is that the tooling or the habit?
3. The goal was "resumable" and one of four tickets did not land — is the goal met?
```

## Edge cases
- **No tickets supplied.** There is no sheet without them. Say so and stop.
- **Tickets with no transitions.** They are counted in delivered/carried totals but excluded from cycle time. Name them, and print the `n` you actually computed on, as both examples do. If more than half the delivered tickets have no transitions, do not report cycle time at all: say the board does not record state changes, and that the timing half of this sheet is unavailable.
- **Fewer than 8 delivered tickets.** Do not report p85. At n=5 the 85th percentile falls between the 4th and 5th of 5 values, so it is the slowest ticket wearing a statistic's name. Report `n`, the range, and the individual outlier instead, as the second example does. [judgment, anchored on the arithmetic: p85 index = 0.85·(n−1)+1]
- **Fewer than 5 delivered tickets.** No median either. At n=3 the median is one ticket and at n=4 it is the average of two, so the word adds authority the number has not earned. List every ticket with its duration and the range, as the second example does, and let the room read them.
- **The board does not record what was committed at sprint start.** The committed-vs-added split is the first count in the process and it collapses without this. Report one population, say explicitly that commitment data was unavailable, and drop question 1's framing — you cannot ask what gets in when you cannot see what was planned.
- **No previous retro actions supplied.** Write `Last retro's actions: none recorded` and move on. Do not treat the absence as a finding unless the team says they keep them.
- **Previous actions written as prose, not items.** Split the paragraph into the actions it actually contains, quote each one, and verdict them separately. If a sentence contains no checkable commitment, it is `not checkable` — say so rather than dropping it.
- **No sprint goal.** First line of the evidence block, as in the first example, and it usually becomes one of the three questions.
- **A sprint with 200+ tickets** (a merged board, or a quarter passed off as a sprint). Do not summarise all of it. Report the counts, then compute cycle time on the delivered set only and say so; if that is still over 200, take the 200 most recently delivered and state the cut.
- **Malformed or duplicated transitions** (a ticket moved to "In progress" four times, or timestamps out of order). Use the first entry to "In progress" and the first entry to "Done"; if "Done" precedes "In progress", exclude the ticket and list it with the others that have no usable timing.
- **A ticket delivered but reopened before sprint end.** It is carried over, not delivered. Say which tickets this rule moved.
- **No PRs supplied, or none link to a ticket.** Time in review is the second half of step 2 and it is simply unavailable. Write `not measured` with the reason, as the small-sprint example does, and do not substitute a proxy such as time in a "In Review" board state without saying that is what you did.
- **Sub-tasks and epics in the ticket list.** Count leaf tickets only. An epic delivered alongside its five children is one thing shipped, not six, and counting both inflates every number on the sheet. Say how many rows you excluded as parents.
- **The sprint has no end date, or the range spans several sprints.** Ask for the boundary before counting. Committed-vs-added is defined against a sprint start; without one, both populations are guesses.
- **Two of the three questions would come from the same observation.** Take the next candidate down the ranked list. Three questions about review latency is one question printed three times.

## License
MIT
