---
name: sprint-retro-facilitator
owner: opsmith
category: Ticket ops
description: Prepares a retrospective from the sprint's data — what shipped, what slipped, where time went — and turns it into three concrete questions for the team.
version: v2
license: MIT
updated: 2026-08-28
recommended: false
security_checked: true
url: https://emdly.com/skills/opsmith/sprint-retro-facilitator
raw: https://emdly.com/raw/opsmith/sprint-retro-facilitator.md
install: npx @emdly/cli add opsmith/sprint-retro-facilitator
---

# Sprint retro facilitator

A retro is only useful when it starts from facts. This skill turns the sprint's tickets and PRs into a short evidence sheet and three questions worth an hour of the team's time.

## When to use
- The last day of a sprint, before the retro is scheduled.
- After a release that went badly, over any date range.

## Input
Tickets in the sprint with their transitions and timestamps, PRs with open/merge times, the sprint goal, and — if the team keeps them — the previous retro's action items.

## Process
1. **Count honestly.** Committed vs. delivered, added mid-sprint, carried over. Carry-overs that were also carried over last sprint get their own list: *slipped twice*.
2. **Where the time went.** Cycle time per ticket (first "In progress" → "Done"), reported as median and 85th percentile. Time in review separately.
3. **Previous actions.** For each action item from the last retro: done, partly, not started — with the evidence.
4. **Three questions.** Pick the three observations with the largest gap between plan and reality and phrase each as a neutral question the team can answer.

## Rules
- Facts and questions only. Never assign blame, never name who was slow — cycle time is a team number.
- Do not propose solutions; that is the team's job in the room.
- If the sprint goal is missing, say so as the first observation. That is usually the finding.
- Keep the sheet to one screen.

## Output format
```
## Sprint 34 — evidence
- Committed 21 · delivered 14 · added mid-sprint 6 · carried over 7 (slipped twice: PROJ-88, PROJ-91)
- Cycle time: median 2.4 d · p85 6.1 d · time in review median 1.3 d
- Last retro's actions: "pair on flaky tests" done · "limit WIP to 2" not started

## Three questions
1. Six tickets arrived mid-sprint and four of the carry-overs are from the original plan. What decides what gets in?
2. Review is more than half of cycle time. What is the review waiting on?
3. PROJ-88 has slipped twice. What is different about it?
```

## License
MIT
