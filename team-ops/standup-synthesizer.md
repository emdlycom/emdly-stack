---
name: standup-synthesizer
owner: opsmith
category: Team ops
description: Reads yesterday's commits, tickets and threads, and writes each person's standup draft — blockers first, no ceremony.
version: v5
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
- **The budget is 40 lines of person blocks** [house rule], and the team size decides the mode. The footer and the mode header sit outside the budget. Each mode's line count is a maximum, so a block that comes out shorter (`on leave`) never pushes you over.

  | mode | team size | lines per person | worst case |
  |---|---|---|---|
  | full | 1–8 | 5: name, Blocked, Yesterday, Today, blank | 8 × 5 = 40 |
  | compact | 9–20 | 2: name-and-Blocked, then Yesterday/Today | 20 × 2 = 40 |
  | blockers-only | 21+ | blocked people in full, then two roll-up lines | fixed, ~12 |

  The band boundaries are the arithmetic, not a preference: 9 × 5 = 45 breaks the budget, which is why nine people cannot use full mode, and 21 × 2 = 42 breaks it again, which is why twenty-one cannot use compact. Say at the top which mode you used and show the arithmetic, exactly as the examples do.
- Never rank people, count their items, or comment on volume. This is a status list, not a performance record. No "busiest", no totals per person, no ordering by output.
- A blocker that cleared inside the window is not a blocker. It goes to Yesterday as "unblocked X", so the team can see it moved without it holding the top of the list.
- Do not editorialise. No "still", "again", "finally", "as usual". These read as a judgement on a person, which is the one thing this output must not be.

## Output format

**Full mode — 8 people or fewer.** Five lines each.

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
Four people at five lines each is 20, well inside the 40-line budget; the two footer lines sit outside it. Eight people is 8 × 5 = 40, which is the last size full mode fits.

**Compact mode — 9 to 20 people.** Two lines each, no blank line between people. Blocked stays on the name line so blockers are still readable in one scan.

```
## Standup — 11 people, compact mode
Full blocks omitted: 11 × 5 = 55 lines exceeds the 40-line budget. Compact is 11 × 2 = 22.

**Dev** · Blocked: waiting on review: #431, requested 2026-08-26, no review since
  **Yesterday:** opened #431 (rate limiter) · **Today:** PROJ-118 spike
**Mara** · Blocked: staging DB refresh, waiting on @tomasz since 2026-08-26 (thread: "staging snapshot", #platform)
  **Yesterday:** merged retry backoff (#418) · moved PROJ-92 to QA · **Today:** billing migration review
**Priya** · Blocked: pricing copy sign-off, waiting on: (not stated)
  **Yesterday:** shipped PROJ-77 (opened and merged #421) · **Today:** (not stated)
**Aditi** · Blocked: —
  **Yesterday:** merged #427 (feature-flag cleanup) · **Today:** PROJ-121
**Ben** · Blocked: —
  **Yesterday:** reviewed PR #412 · moved PROJ-88 to Done · **Today:** (not stated)
**Chen** · Blocked: —
  **Yesterday:** no recorded activity · **Today:** (not stated)
**Elena** · Blocked: —
  **Yesterday:** opened #429 (search index rebuild) · **Today:** finishing #429
**Sam** · Blocked: —
  **Yesterday:** no recorded activity · **Today:** (not stated)
**Tomasz** · Blocked: —
  **Yesterday:** opened #423 (webhook retries) · thread: (title withheld), #people-ops · **Today:** staging DB refresh (blocking @mara)
**Wei** · on leave
**Yuki** · Blocked: —
  **Yesterday:** unblocked OPS-55 (Stripe sandbox recovered) · **Today:** OPS-61

Sources: commits/PRs 118 rows · ticket transitions 44 rows · threads 17 rows.
Window: 2026-08-28 09:00 → 2026-08-29 09:00.
```
Three blocked, eight unblocked, eleven in all. Ten of them take two lines (10 × 2 = 20) and Wei's `on leave` block takes one, so 21 lines against a 22-line worst case. Both sit inside the 40-line budget, and the two footer lines sit outside it.

**Blockers-only mode — 21 people or more.**
```
## Standup — 22 people, blockers-only mode
Full blocks omitted: 22 × 5 = 110 lines, and compact 22 × 2 = 44 lines, both exceed the
40-line budget.

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

## Edge cases
- **No input at all.** Say "no activity data supplied" and stop. Do not produce a template with empty names.
- **A person has no rows.** They still get a block: `Blocked: —`, `Yesterday: no recorded activity`, `Today: (not stated)`. Never write "worked on the usual".
- **The whole team has no rows** (holiday, outage in the data source). Say which sources returned zero and ask whether the window is right, before printing eight identical empty blocks. On an unattended run this is a stop, not an output.
- **One source is unavailable** — no ticket access, no thread access, git only. The method does not collapse, but it narrows: blockers are the point of this skill and most blockers live in threads and ticket states. Produce the standup from what you have, and name the missing source in the footer, as the example does. Never let a missing source read as an empty day. Two or more sources unavailable is a stop.
- **Malformed timestamps or a mixed timezone.** If more than one timezone appears and none is declared, state the window in UTC and say you did. If a timestamp will not parse, keep the item and mark it `(time unknown)` rather than dropping it — a dropped item looks like an idle person.
- **Too much activity for one person.** Over 20 items after collapsing, keep the 5 most recent and write `+ 14 more items, not listed` with the real remainder. Never silently truncate.
- **Nine to twenty people.** Full mode is impossible: 9 × 5 = 45 lines already breaks the 40-line budget. Use compact mode, exactly as the second example shows, and print the arithmetic at the top. Do not shorten the blocks freehand and do not go straight to blockers-only — compact still carries everyone's Yesterday and Today, and dropping them at nine people is a bigger loss than the two lines it saves.
- **Twenty-one people or more.** Compact breaks too: 21 × 2 = 42. Switch to blockers-only mode, exactly as the third example shows: the blocked people in full, then two roll-up lines naming everyone else. Say at the top which mode you used and why. The counts on the three lines must add to the team size.
- **A name in the input that is not on the team.** Include them. A contractor or a partner engineer showing up in the data is information, not noise. Do not silently drop a person because you do not recognise the handle. They count toward the team size that picks the mode.
- **Reverted commits** count once, as "reverted X". The original and the revert are not two items.
- **A PR opened and merged the same day** is one item: "shipped X".
- **One PR, two people** (co-authored, or one opened it and another merged it). It appears once under each, described from that person's side: "opened #423" and "merged #423". Do not pick a single owner.
- **A blocker that is also someone else's item.** It appears in both blocks — once as the block, once as the work. Do not deduplicate across people.
- **Weekends and holidays.** The window spans back to the last working day at 09:00, so Monday reaches Friday morning. If the input covers a shorter span than the window, say the span you actually got.
- **A blocker that has been open for more than one window.** Say how long: `waiting on @tomasz since 2026-08-26`. Age is the only thing that distinguishes a blocker worth the meeting's time, and it comes from the timestamp, not from a judgement.
- **Someone is on leave.** If the input marks it, write `on leave` and nothing else for them. If it does not, they read as `no recorded activity`, which is accurate — do not guess at leave.

## Stop and hand back

This skill runs on a schedule and posts to a channel with nobody reading it first, and every line of it names a person and their open work. Stop means: post nothing to the channel, and send what you have to the person who owns the scheduled job. It is not a caution added to a standup that goes out anyway.

- **Every source returned zero rows.** Do not post. A standup saying the whole team did nothing is almost always a broken query or a wrong window, and posting it costs more trust than skipping a day. Send the window, the three row counts and any query error to the job owner, who decides whether to re-run or skip the day.
- **Two or more of the three sources errored.** Do not post. Blockers live in threads and ticket states, so a git-only standup reads as "no blockers" when the truth is that nobody knows. Send the errors to the job owner. One source down is not a stop: post it with the missing source named in the footer, as the full-mode example does.
- **A thread or ticket title that names a person alongside a condition, a salary figure, a legal term, or a disciplinary word.** Withhold the title, per the rules. If withholding it leaves the item meaningless, drop the item from the channel post and send it to the person whose item it is. They decide whether it belongs in standup.
- **A blocker whose named counterpart is a person and whose evidence reads as a complaint about them** rather than a wait — an @-mention that escalates, a thread title naming someone next to a failure. Do not post it as the top line of a channel message. Hand it to whoever runs the standup, with the evidence quoted, and let them raise it in the room.
- **The same person reads `no recorded activity` for three consecutive windows** with no leave marker in the input. Do not let a public run of empty blocks stand in for a performance record, which the rules forbid. Tell the job owner once, privately, and keep posting the block unchanged.
- **The target channel is customer-facing, shared with a client, or otherwise outside the team.** This output names individuals, their open work and who they are waiting on. Do not post. Ask whoever configured the job to confirm the channel before the next run.
- **The input contains message bodies rather than thread titles.** The Input section asks for titles only. Do not post from bodies and do not summarise them. Hand back to the job owner to fix the query; private message content posted to a channel cannot be recalled.

## License
MIT
