---
name: standup-synthesizer
owner: opsmith
category: Ticket ops
description: Reads yesterday's commits, tickets and threads, and writes each person's standup draft — blockers first, no ceremony.
version: v4
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/opsmith/standup-synthesizer
raw: https://emdly.com/raw/opsmith/standup-synthesizer.md
install: npx @emdly/cli add opsmith/standup-synthesizer
---

# Standup synthesizer

Turn a day of activity into a standup people can read in thirty seconds. The goal is not a report — it is to surface blockers before the meeting so the meeting can be short. Everything in the output comes from recorded activity; where there is no activity, the output says so rather than filling the gap with plausible work.

## When to use
- Every morning before standup, on a schedule, with a Slack or e-mail trigger.
- In Claude Code when someone asks "what did I do yesterday".
- After a long weekend or a holiday, when nobody remembers where things stopped.
- Before a handover, when one person needs another's open threads in writing.

## Input
Per person, for the window:
- **Commits and PRs** — title, branch, state (open / merged / closed), reviewers requested, review comments left.
- **Ticket transitions** — ticket id, from-state, to-state, timestamp.
- **Threads they wrote in** — thread title and channel only. Never the message bodies.
- **The window itself** — default: since the previous working day at 09:00 in the team's timezone. [house rule; there is no standard here, and 09:00 is chosen so that late-evening work lands in the day it was done.]

A person with no rows in any of the three sources is still listed. Absence is a finding.

> Thresholds above are defaults; report the thresholds you used.

## Five passes
1. **Collect.** Pull every row in the window, per person, and record the row count from each source — you report those counts in the footer and you need them for the edge cases below. Identity is matched on the handle the input gives; if a git author email and a chat handle do not obviously belong to the same person, do not merge them, list both, and say you did. Skip bot and automation authors (dependabot, release bots, CI) entirely; they are not in standup.
2. **Classify.** Every item is Blocked, Yesterday, or Today. Blocked means waiting on something outside the person's control, and only these five things count:

   | blocker | evidence in the input | how it is written |
   |---|---|---|
   | another person | a review request open > 1 working day, or an @-mention asking for something | `waiting on @name` |
   | a decision | a thread with a question and no answer in the window | `waiting on a decision (thread: "title", #channel)` |
   | an environment | a ticket in a blocked/on-hold state, or a failed deploy | `waiting on: staging` |
   | a review | a PR open with reviewers requested and no review in the window | `waiting on review: #418` |
   | an external vendor | a ticket or thread naming a third party | `waiting on: <vendor>` |

   Nothing else is a blocker. Difficulty is not a blocker. Being busy is not a blocker. A PR opened this morning is not yet waiting on review.
   **Yesterday** is work with a completed state in the window: a merged PR, a ticket that moved to a done-side state, a review left. **Today** is populated from exactly three sources, in this order: a ticket the person moved into an in-progress state and has not moved out of; a PR of theirs still open with unresolved review comments; a plan they wrote in a thread in the window. If none of the three exists, Today is `(not stated)`. Yesterday's finished work is never carried into Today.
3. **Collapse.** Five commits on one branch become one item. Ten review comments on one PR become "reviewed PR #412". Three transitions on one ticket become the last one. Collapse within a person and within a day; never across people.
4. **Order.** People with blockers first, then alphabetically by the name as it appears in the input. Within a person: Blocked, Yesterday, Today. Within a line, order items by their timestamp, oldest first, so the line reads as the day happened.
5. **Footer.** Print the row count from each source and the window you actually used, as the examples show. Anyone reading the standup should be able to tell an empty day from an empty query, and the footer is the only thing that distinguishes them.

## Rules
- **Blockers first**, and each blocker names who or what it waits on. A blocker with no named counterpart is written `waiting on: (not stated)` — that is itself the thing standup should fix.
- Only report what the input shows. No activity means `no recorded activity`. No stated plan means `(not stated)`. Never infer today's plan from yesterday's work.
- One line per item, verb first. Past tense for done, present participle for in progress: "Merged the retry backoff PR", "Reviewing the billing migration".
- Drop these words wherever they appear: just, basically, quick, small, simply, actually, a bit, some. Closed list.
- Never include private thread content. Reference a thread by its title and channel only. If the title itself contains something that reads as private (a person's name plus a condition, a salary figure, a legal term), write `thread: (title withheld)`.
- Five lines per person maximum: the name, Blocked, Yesterday, Today, and one blank. Eight people is 40 lines; that is the whole standup. [house rule]
- Never rank people, count their items, or comment on volume. This is a status list, not a performance record. No "busiest", no totals per person, no ordering by output.
- A blocker that cleared inside the window is not a blocker. It goes to Yesterday as "unblocked X", so the team can see it moved without it holding the top of the list.
- Do not editorialise. No "still", "again", "finally", "as usual". These read as a judgement on a person, which is the one thing this output must not be.

## Output format
```
### Mara
**Blocked:** staging DB refresh — waiting on @tomasz (thread: "staging snapshot", #platform)
**Yesterday:** merged retry backoff (#418) · moved PROJ-92 to QA · reviewed PR #412
**Today:** billing migration review · PROJ-104 spike

### Priya
**Blocked:** pricing copy sign-off — waiting on: (not stated)
**Yesterday:** shipped PROJ-77 (opened and merged #421) · reverted #419
**Today:** (not stated)

### Sam
**Blocked:** —
**Yesterday:** no recorded activity
**Today:** (not stated)

### Tomasz
**Blocked:** —
**Yesterday:** opened #423 (webhook retries) · thread: (title withheld), #people-ops
**Today:** staging DB refresh (blocking @mara) · finishing #423

Sources: commits/PRs 47 rows · ticket transitions 22 rows · threads 9 rows.
Window: 2026-08-28 09:00 → 2026-08-29 09:00. Ticket transitions for Sam were
unavailable (Jira 503); his line reflects git and threads only.
```
Four people at five lines each is 20, plus the source footer. Eight people is 40.

Above fifteen people, five lines each will not fit. Switch to blockers-only and say so:
```
## Standup — 22 people, blockers-only mode
Full blocks omitted: 22 people × 5 lines exceeds the 40-line limit.

**Blocked (4 of 22)**
- Mara — staging DB refresh, waiting on @tomasz (thread: "staging snapshot", #platform)
- Priya — pricing copy sign-off, waiting on: (not stated)
- Dev — waiting on review: #431, requested 2026-08-26, no review since
- Yuki — waiting on: Stripe (ticket OPS-55, sandbox 500s since 2026-08-27)

**No blockers recorded (16)**: Aditi, Ben, Chen, Elena, Felix, Grace, Hana, Ivan,
Jonas, Kim, Lena, Marco, Nina, Omar, Rosa, Tomasz
**No recorded activity (2)**: Sam, Wei

Sources: commits/PRs 214 rows · ticket transitions 88 rows · threads 31 rows.
Window: 2026-08-28 09:00 → 2026-08-29 09:00.
```
4 + 16 + 2 = 22. Every person is accounted for in exactly one of the three lines.

## Running it unattended
On a schedule, with nobody reading before it posts, two rules change. Post nothing when every source returned zero rows — a standup that says the whole team did nothing is almost always a broken query, and posting it costs more trust than skipping a day. Send the failure to whoever owns the job instead. And never post a partial run silently: if a source errored, the footer says which one, in the message that goes to the channel, not in a log.

## Edge cases
- **No input at all.** Say "no activity data supplied" and stop. Do not produce a template with empty names.
- **A person has no rows.** They still get a block: `Blocked: —`, `Yesterday: no recorded activity`, `Today: (not stated)`. Never write "worked on the usual".
- **The whole team has no rows** (holiday, outage in the data source). Say which sources returned zero and ask whether the window is right, before printing eight identical empty blocks.
- **One source is unavailable** — no ticket access, no thread access, git only. The method does not collapse, but it narrows: blockers are the point of this skill and most blockers live in threads and ticket states. Produce the standup from what you have, and name the missing source in the footer, as the example does. Never let a missing source read as an empty day.
- **Malformed timestamps or a mixed timezone.** If more than one timezone appears and none is declared, state the window in UTC and say you did. If a timestamp will not parse, keep the item and mark it `(time unknown)` rather than dropping it — a dropped item looks like an idle person.
- **Too much activity for one person.** Over 20 items after collapsing, keep the 5 most recent and write `+ 14 more items, not listed` with the real remainder. Never silently truncate.
- **Too many people.** Over 15, five lines each breaks the 40-line rule. Switch to blockers-only mode, exactly as the second example shows: the blocked people in full, then two roll-up lines naming everyone else. Say at the top which mode you used and why. The counts on the three lines must add to the team size.
- **A name in the input that is not on the team.** Include them. A contractor or a partner engineer showing up in the data is information, not noise. Do not silently drop a person because you do not recognise the handle.
- **Reverted commits** count once, as "reverted X". The original and the revert are not two items.
- **A PR opened and merged the same day** is one item: "shipped X".
- **One PR, two people** (co-authored, or one opened it and another merged it). It appears once under each, described from that person's side: "opened #423" and "merged #423". Do not pick a single owner.
- **A blocker that is also someone else's item.** It appears in both blocks — once as the block, once as the work. Do not deduplicate across people.
- **Weekends and holidays.** The window spans back to the last working day at 09:00, so Monday reaches Friday morning. If the input covers a shorter span than the window, say the span you actually got.
- **A blocker that has been open for more than one window.** Say how long: `waiting on @tomasz since 2026-08-26`. Age is the only thing that distinguishes a blocker worth the meeting's time, and it comes from the timestamp, not from a judgement.
- **Someone is on leave.** If the input marks it, write `on leave` and nothing else for them. If it does not, they read as `no recorded activity`, which is accurate — do not guess at leave.

## License
MIT
