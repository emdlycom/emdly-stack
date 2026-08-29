---
name: standup-synthesizer
owner: opsmith
category: Ticket ops
description: Reads yesterday's commits, tickets and threads, and writes each person's standup draft — blockers first, no ceremony.
version: v3
license: MIT
updated: 2026-08-27
recommended: false
security_checked: true
url: https://emdly.com/skills/opsmith/standup-synthesizer
raw: https://emdly.com/raw/opsmith/standup-synthesizer.md
install: npx @emdly/cli add opsmith/standup-synthesizer
---

# Standup synthesizer

Turn a day of activity into a standup people can read in thirty seconds. The goal is not a report — it is to surface blockers before the meeting so the meeting can be short.

## When to use
- Every morning before standup, on a schedule, with a Slack or e-mail trigger.
- In Claude Code when someone asks "what did I do yesterday".

## Input
For each person: commits and PRs (title, state, reviewers), ticket transitions, and threads they wrote in. A time window (default: since the previous working day 09:00).

## Rules
- **Blockers first.** Anything waiting on another person, a decision, an environment or a review goes to the top with who it waits on.
- Only report what the input shows. No activity means "no recorded activity" — never fill the gap with plausible work.
- One line per item, verb first, past tense for done, present for in progress: "Merged the retry backoff PR", "Reviewing the billing migration".
- Collapse noise: five commits on one branch are one item. Ten review comments are "reviewed PR #412".
- Drop ceremony words ("just", "basically", "quick", "small").
- Never include private thread content; reference the thread by title only.

## Output format
```
### Mara
**Blocked:** staging DB refresh — waiting on @tomasz (thread: "staging snapshot")
**Yesterday:** merged retry backoff (#418) · moved PROJ-92 to QA
**Today:** billing migration review · PROJ-104 spike

### Tomasz
**Blocked:** —
**Yesterday:** no recorded activity
**Today:** (not stated)
```
Order people with blockers first, then alphabetically. Whole standup under 40 lines for a team of eight.

## Edge cases
- Reverted commits count once, as "reverted X" — the original and the revert are not two items.
- A PR opened and merged the same day is one item: "shipped X".
- Weekends: the window spans back to Friday morning.

## License
MIT
