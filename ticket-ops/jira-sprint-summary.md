---
name: jira-sprint-summary
owner: launifycorp
category: Ticket ops
description: The outward-facing report after a sprint closes: a verdict against the sprint goal, committed versus delivered, blockers with a measured duration and a named dependency, hours grouped by work area with coverage stated first, and at most three improvements that each cite a number. Never per person.
version: v1
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/jira-sprint-summary
raw: https://emdly.com/raw/launifycorp/jira-sprint-summary.md
install: npx @emdly/cli add launifycorp/jira-sprint-summary
---

# Jira sprint summary

The report that goes out after the sprint closes, to the people who were not in it. It says what shipped against what was promised, what actually blocked the work and for how long, where the hours went, and the two or three things worth changing. It is an assessment, and it says so.

Two things it will not do, because doing them badly is worse than not doing them. It does not attach hours or verdicts to a person: Jira worklogs are a logging habit, not a timesheet, and a per-person number from patchy data reads as a measurement when it is an artefact. And it does not call anything a blocker without a clock and a dependency on it.

Related but different: `opsmith/sprint-retro-facilitator` prepares an evidence sheet and three questions *for the team's own retro*, and deliberately refuses to diagnose or recommend. This one is the outward-facing summary, and it does both. If the audience is the team in a room, use that one instead.

## When to use

- The sprint closed and a lead, a stakeholder or a steering group needs to know how it went.
- A monthly or quarterly roll-up over several sprints, for the same audience.
- The sprint went badly and someone is going to ask why before the retro happens.
- Handover to a lead who was not there.

Not for: a retro sheet (see above), a live standup (`opsmith/standup-synthesizer`), or scoring tickets before the sprint (`opsmith/jira-ticket-scorer`).

## Input

Required:

- **Issues in the sprint**, each with: key, type, status, the sprint field, and the changelog of status transitions with timestamps. Without transitions there is no blocker analysis and no durations.
- **The sprint boundary** — start and end timestamps. Committed-versus-added is defined against the start; without it, both populations are guesses.
- **The sprint goal**, in the words the team wrote it. The verdict is assessed against this, not against ticket count.

Used when present, named as absent when not:

- **Worklogs** — `timespent` per issue, or the individual worklog entries. Without them the Hours section does not run.
- **Original estimates** — `timeoriginalestimate`. Needed for the estimate-versus-actual distribution, and for nothing else.
- **Issue links**, specifically `is blocked by`, and the **Flagged** field with the timestamps it was set and cleared.
- **The previous sprint's carry-over list**, to tell a first slip from a third.

## Verdict

One line, first line, assessed against the sprint goal:

| verdict | when |
|---|---|
| `MET` | the goal's work shipped inside the sprint |
| `MET, SCOPE CHANGED` | the goal shipped, but the committed set was materially rewritten to get there — say what was dropped |
| `MISSED` | the goal's work did not ship, whatever else did |
| `NO GOAL STATED` | nothing to assess against, and that is the finding |

A sprint that delivered twenty tickets and not the goal is `MISSED`. Ticket counts go in the body, not in the verdict.

## Section 1 — Delivered

Two populations, kept apart and never merged: **committed at start** and **added mid-sprint**. For each, delivered and carried. Both identities must hold — `committed = delivered + carried`, `added = delivered + carried` — and if either does not, the input is inconsistent and you say so instead of publishing a total.

Carry-overs that also appear in the previous sprint's carry-over list get named as repeat slips, with the count.

A ticket delivered and then reopened before sprint end is carried, not delivered. Say which tickets that rule moved.

Count leaf issues only. An epic delivered with its five children is one thing shipped; counting both inflates every number below it. Report how many parent rows you excluded.

## Section 2 — Blockers

A blocker needs a **duration** and a **thing it was waiting on**. "It was blocked" without a clock is a feeling, and it is the sentence this section exists to replace.

Five evidence sources, in descending order of how much they are worth:

| source | what it gives you |
|---|---|
| `Flagged` field with set and cleared timestamps | a measured duration, the strongest evidence available |
| `is blocked by` link to an issue not resolved inside the window | a named dependency and a duration from the link |
| time parked in a waiting state (Blocked, On Hold, Waiting for X) | a measured duration from the changelog |
| an open issue whose last transition is older than the stall threshold | a duration, but the cause is unknown — say so |
| a comment saying the work is blocked | quote it, mark it **asserted**, never **measured** |

Classify each as **internal** (the team could have cleared it) or **external** (another team, a vendor, a customer, an approval). The distinction decides whether Section 4 can propose anything: an improvement aimed at an external blocker is a request to someone else, not a team action.

Stall threshold: no transition for **3 working days** on an issue still open. [house rule — it is roughly half a two-week sprint, long enough that a normal in-progress ticket does not trip it]

> Thresholds above are defaults; report the thresholds you used.

## Section 3 — Hours

Coverage first, hours second. A total that omits the coverage line is misleading and this skill does not print one.

**Coverage** is the share of delivered issues carrying any logged time. Report it as a fraction and a percentage before any hours figure.

**Never sum `timespent` and `aggregatetimespent` in the same total.** The aggregate field rolls sub-task time up into the parent, so mixing them counts sub-task work twice. Pick one basis, state which, and apply it to every row.

**Never reconstruct hours from status durations.** Time in an in-progress state is calendar time with nights and weekends in it, not effort. If worklogs are missing, the Hours section reports `not available`, and the reason.

Group hours by **area of work** — epic, goal, or issue type. Never by person. Always report unplanned work as its own line, because it is the number that explains the sprint.

**Estimate versus actual** runs only on issues carrying both an original estimate and logged time, and only when coverage clears the floor. Report it as a distribution with `n`, not as a single accuracy ratio: a median ratio and the count over and under. One number implies a precision that a habit-driven field cannot support.

Coverage floor for estimate-versus-actual: **60% of delivered issues**. [judgment — below that, the tickets that happen to be logged are a self-selected sample, usually the ones someone was already tracking closely, so the ratio describes them and not the sprint] Under the floor, print the coverage and the refusal, not the ratio.

## Section 4 — Where to improve

At most three, ranked by the size of the gap between plan and reality. Every one of them must carry:

1. **The number it came from.** No observation without a figure already printed above it.
2. **Inside or outside the team's control**, following the internal/external split from Section 2.
3. **A change to a process, a policy, or a sequence** — never to a person, and never "communicate better".

A finding whose only available action is outside the team is still reported, phrased as the request it is, and addressed to the group that owns it.

If fewer than three survive that test, print fewer. Padding the list to three teaches the reader to skim it.

## Rules

- **Facts before assessment, and visibly separated.** Sections 1 to 3 are counts. Section 4 is judgment, and the reader must be able to see where one stops.
- **Nothing per person.** No hours, ticket counts, cycle times, review counts or verdicts attached to a name. People appear as the owner of a follow-up action, and nowhere else.
- **Every percentage carries its two numbers** in brackets, so the reader can check it without the source data.
- **Say what was not measured.** A section that could not run prints `not available` and the missing input. Silence reads as zero.
- **Story points, if reported at all, are reported alone** and never converted to hours or set beside them. They are an estimate of relative size, and putting them next to logged time invites arithmetic that means nothing.
- **Under 60 lines** for a single sprint, excluding the appendix of excluded issues. [house rule — this is read by someone with ten minutes]

## Output format

```
SPRINT 42 · Checkout rebuild · 12 Aug – 26 Aug

VERDICT  MET, SCOPE CHANGED
Goal: "guest checkout live behind a flag for 10% of traffic." Shipped 25 Aug.
Four committed tickets were dropped mid-sprint to get there — PROJ-611, 612,
619, 627, all importer work, all moved to Sprint 43.

DELIVERED
Committed   18 · delivered 12 · carried 6
Added       5  · delivered 4  · carried 1
Total       23 · delivered 16 · carried 7
Repeat slips: PROJ-588 (3rd sprint), PROJ-591 (2nd)
Excluded as parents: 3 epics
Moved to carried by the reopen rule: PROJ-634

BLOCKERS
1. PROJ-620 · external · 6.4 d · Flagged 14 Aug 09:12 → 20 Aug 16:40
   Waiting on payment provider sandbox credentials. Measured.
2. PROJ-618 · internal · 4.1 d · is blocked by PROJ-617 (delivered 22 Aug)
   Schema migration had to land first. Measured.
3. PROJ-629 · unknown · 5.2 d · no transition since 19 Aug, still open
   Stalled past the 3-working-day threshold. Cause not in the data.
Asserted but not measured: PROJ-624 — comment 18 Aug, "blocked on design",
no Flagged field set and no link. Not counted in the durations above.

HOURS
Coverage 13 of 16 delivered issues logged time [81.2%]. Basis: timespent, leaf
issues only.
Total logged 214.5 h
  Checkout epic          92.0 h  [42.9%]
  Importer               61.5 h  [28.7%]
  Unplanned support      38.0 h  [17.7%]
  Maintenance            23.0 h  [10.7%]
Estimate vs actual: n=9 of 16 (issues with both fields). Median ratio 1.4×
actual to estimate. 7 over, 2 under. Largest 3.2× (PROJ-620, the blocked one).

WHERE TO IMPROVE
1. Unplanned support took 38.0 h [38.0 of 214.5 logged, 17.7%] and nobody
   planned for it. Inside the team's control. Reserve a share of the next
   sprint rather than discovering it, and see whether 17.7% holds a second time.
2. PROJ-620 waited 6.4 days on external credentials, and the estimate on it
   missed by 3.2×. Outside the team's control. Request to platform: get sandbox
   credentials issued before the sprint that needs them starts.
3. PROJ-588 has now slipped three sprints. Inside the team's control. Either it
   gets an owner and lands in Sprint 43, or it leaves the board.

NOT MEASURED
3 delivered issues logged no time — excluded from Hours, counted in Delivered.
Story points not reported: field not supplied.
```

Thin data:

```
SPRINT 43 · no goal recorded · 27 Aug – 09 Sep

VERDICT  NO GOAL STATED
Nothing here can be assessed against intent. This is the finding.

DELIVERED
Committed   9 · delivered 7 · carried 2
Added       0 · delivered 0 · carried 0
Repeat slips: not determinable (no previous carry-over list supplied)

BLOCKERS
None evidenced. The Flagged field is not enabled on this board, no issue
carries an "is blocked by" link, and no issue stalled past the threshold.
This means none were recorded, not that none happened.

HOURS
not available — no worklogs supplied. Not reconstructed from status durations,
because that measures calendar time and not effort.

WHERE TO IMPROVE
1. The sprint ran without a recorded goal, so success is undefined for everyone
   outside the team. Inside the team's control: write one on the board at
   planning, in a sentence.
Only one observation cleared the evidence bar. Two are not printed.
```

The refusal:

```
NOT SUMMARISED · Sprint 44

Blocked: issues supplied without changelogs, and no sprint start timestamp.

Without transitions there are no durations, so Blockers cannot run and the
reopen rule cannot be applied. Without a start timestamp the committed set
cannot be separated from mid-sprint additions, and every count in Delivered
would silently merge the two.

What would be produced instead is a list of ticket titles with a total next to
it, which reads like a measurement and is not one.

Needed: the changelog for each issue, and the sprint start and end timestamps.
```

## Edge cases

- **No sprint goal.** Verdict is `NO GOAL STATED`, first line, and it becomes the first improvement item. Do not infer a goal from the tickets — that is writing the goal after the fact and grading against it.
- **The board does not record what was committed at start.** Report one population, say commitment data was unavailable, and drop the scope-change branch of the verdict. `MET, SCOPE CHANGED` is unreachable without it.
- **No worklogs.** Hours reports `not available`. Do not substitute status durations, and do not fall back to story points.
- **Coverage under the floor.** Print coverage and the total. Print no estimate-versus-actual ratio, and say which floor stopped it.
- **Worklogs logged outside the sprint window** on a sprint issue — a Monday catch-up for the previous Friday. Count them against the issue and note the window mismatch; do not silently redistribute them.
- **Mixed `timespent` and `aggregatetimespent` in the input.** Choose the leaf-level basis, state it, and report how many parent rows you dropped to avoid double counting.
- **The Flagged field is not enabled.** Say so explicitly, as the thin example does. Absence of flags is not absence of blockers, and a reader will otherwise assume the sprint ran clean.
- **A blocker asserted only in a comment.** Quote it, mark it asserted, and keep it out of the measured durations. It still belongs in the report.
- **An issue blocked by another issue in the same sprint.** Internal, and it is a sequencing finding, not a dependency finding. Say which one had to land first.
- **A sprint with no carried issues and no blockers.** Report it plainly. Do not manufacture a finding to fill Section 4.
- **Several sprints in one input.** Run Sections 1 to 3 per sprint, one row each, then run Section 4 once across the range. Do not average medians; pool the issues and compute once, and say that is what you did. Cap at six sprints.
- **Over 300 issues.** Report the counts in full, then run Blockers and Hours on the delivered set only and say so.
- **Issues moved between sprints mid-flight**, so the sprint field holds several values. Attribute the issue to the sprint it was in at the boundary, and list the ones this rule moved.
- **Sub-tasks in the issue list.** Leaf-only counting, as above, with the excluded count printed.
- **A ticket in a language other than the report's.** Quote the summary verbatim; do not translate a ticket title into the report and lose the searchable string.

## Stop and hand back

Stop means the section is not produced and the reason is printed in its place.

1. **The summary is for a performance review, a rating, a PIP or a compensation decision.** Say plainly that this report cannot support that: worklog coverage is a logging habit, hours are not output, and nothing here is normalised for the difficulty of what each person picked up. Route to the manager without producing per-person figures, however the request is phrased.
2. **Hours are wanted for client billing or invoicing.** Worklogs are not an invoice. Finance owns the reconciliation between logged time and billable time, and the two are routinely different on purpose.
3. **A request to name who caused a blocker or a slip.** Report the dependency and the duration. The person is not the finding.
4. **A blocker that is an interpersonal conflict, a grievance or anything else an HR process owns.** Do not summarise it, do not quote the comment, and hand it to the person who owns that process.
5. **A comparison against another team, or against an industry benchmark.** Different boards, different definitions of done, different logging habits. The comparison is arithmetic on incompatible units.
6. **The summary states a date or a scope commitment to a customer.** A report about the past becomes a promise about the future the moment it leaves the building. That sentence needs the owner of the commitment, not the author of the summary.
7. **Coverage is under the floor and the requester wants the accuracy number anyway.** Give the coverage figure and decline the ratio. Producing it on request is how a self-selected sample becomes a quoted statistic.

## License

MIT
