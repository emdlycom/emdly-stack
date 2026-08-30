---
name: jira-ticket-scorer
owner: opsmith
category: Ticket ops
description: Scores every Jira ticket for clarity, scope and effort before it hits the sprint. Flags vague acceptance criteria and suggests a rewrite.
version: v4
license: MIT
updated: 2026-08-30
recommended: true
security_checked: true
url: https://emdly.com/skills/opsmith/jira-ticket-scorer
raw: https://emdly.com/raw/opsmith/jira-ticket-scorer.md
install: npx @emdly/cli add opsmith/jira-ticket-scorer
---

# Jira ticket scorer

Score a ticket **before** it is estimated, so the team argues about the work, not about what the words mean. The score is for the refinement session: three numbers a facilitator can read out in ten seconds, the specific sentence that cost each point, and — where the ticket is weak enough to bounce — a rewrite the reporter would recognise as their own ticket, tightened.

## When to use
- As a step in a pipeline: new ticket → score → route to refinement or back to the reporter.
- Standalone in Claude Code: paste the ticket, get the score and the rewrite.
- On a schedule via the API, sweeping everything created since the last run.
- Before a refinement session, to order the agenda worst-first.
- When estimation stalls on a ticket and nobody can say what it is actually asking for.

## Input
Required:
- The ticket's title and description.
- The issue type (bug, story, task, spike, subtask), because the rubric branches on it.

Optional, and each one changes what can be scored:
- Acceptance criteria, if any. Absent is a finding, not a blocker.
- Labels and linked issues, including whether each link is open or closed.
- Comments. When present, treat the newest comment as the current truth and say so.
- The parent ticket, required for subtasks. Without it a subtask cannot be scored — see Edge cases.
- Attachments by name only. You cannot read a screenshot; a repro that lives only in an image is a missing repro.

Score one ticket per run. Do not score an epic as a ticket.

## Rubric
Score three dimensions from 1 to 5. **Do not average them.** Report each with the sentence, or the absence, that set it.

**Clarity** — could an engineer who has never seen this codebase start without asking a question?
- 5: user, trigger, expected outcome and out-of-scope are all explicit.
- 4: all four present, one of them vague enough to need a single clarifying question.
- 3: the goal is clear, the boundary is not.
- 2: the goal is inferable but stated only once, in passing.
- 1: a title and a sentence.

**Scope** — is this one deliverable?
- 5: one change, one owner, one way to verify.
- 4: one change with a second, smaller change that genuinely cannot ship separately.
- 3: two or three related changes that could ship separately.
- 2: two or more changes with different owners.
- 1: a project wearing a ticket's clothes.

**Effort signal** — does the ticket contain what estimation needs (affected components, data, dependencies)?
- 5: components named, edge cases listed, dependencies linked.
- 4: components named, one of data volume or dependencies missing.
- 3: the area is identifiable but nothing is named.
- 2: one hint, nothing more — a label, a linked ticket with no explanation.
- 1: nothing to estimate from.

### Declared house rules
These are conventions this skill applies, not findings read out of the ticket. Say "house rule" next to each one when it fires, so a reporter can argue with the rule rather than with the score.

- Missing reproduction steps caps **Clarity at 2** on a bug. [house rule]
- A spike with no timebox caps **Scope at 3**. [house rule]
- A ticket whose description is under **20 words** caps **Effort signal at 2**, whatever the title claims. [house rule]
- Any score of **2 or below** triggers the suggested rewrite. [house rule]
- Two or more scores at **2 or below** routes the ticket back to the reporter rather than to refinement. [house rule]

> Thresholds above are defaults; report the thresholds you used.

## Rules
- Never invent acceptance criteria that are not in the ticket. Propose them, and label the proposal `proposed`.
- Flag anything you could not verify instead of guessing. "The reporter probably means…" is a flag, not a fact.
- Subtasks inherit their parent's context: score them against the parent's goal, not in isolation.
- Story points already on the ticket are not evidence of clarity. Ignore them for scoring; mention them only where they contradict the effort signal, and say so in one line.
- Keep the rewrite in the reporter's voice. You are tightening, not redesigning the feature. Keep their nouns, their product terms and their ordering; cut hedges, split run-on requirements, and make the boundary explicit. Do not add a requirement they did not write.
- Quote, do not summarise. Every score below 5 names the words that cost the point, in quotes.
- **The whole result is under 200 words, counting the rewrite.** If it would run over, drop to the two most severe flags and write `(2 of N flags shown)`. Never cut the scores or the rewrite to fit.
- One ticket, one result. Never merge two tickets into one score.

## Output format

A ticket that scores low enough to bounce:

```
## PROJ-1184 — "Fix the export"  (Bug)

## Result
- **Clarity:** 2 — no reproduction steps [house rule: missing repro caps Clarity at 2].
  "it breaks for some users sometimes" is the only symptom given.
- **Scope:** 2 — contains a migration ("add a status column") and a UI change
  ("and grey out the button"), different owners, can ship separately.
- **Effort signal:** 3 — the export job is identifiable from the title; no component,
  no row count, no dependency named.
- **Flags:** "as before" refers to behaviour described nowhere in the ticket or its
  comments; linked PROJ-812 is closed; the repro may be in export-error.png, which
  cannot be read.
- **Route:** back to reporter — two scores at or below 2 [house rule].

## Suggested rewrite (reporter's wording kept; proposed criteria labelled)
Title: CSV export returns an empty file for accounts with more than 5,000 rows

The export runs and downloads, but the file has only the header row. Seen by
two users on 14 Aug, both on accounts over 5,000 rows. Smaller accounts export fine.

Steps: open Reports > Export, choose "All time", click Export.
Expected: a CSV with one row per record.
Actual: a CSV with the header row only, 41 bytes.

Out of scope: the "grey out the button" change — raise separately.

proposed — acceptance criteria (not in the original, confirm with the reporter):
- An account with 10,000 records exports 10,000 rows plus a header.
- An export that fails returns an error, not an empty file.

proposed — unverified: whether "as before" means the pre-July behaviour.
Reporter to confirm.
```

A ticket that passes, with nothing to rewrite:

```
## PROJ-1190 — "Add a rate limit to POST /v1/invites"  (Story)

## Result
- **Clarity:** 5 — user, trigger, expected outcome and an explicit out-of-scope line.
- **Scope:** 5 — one endpoint, one owner, one way to verify.
- **Effort signal:** 4 — component named (api/invites), dependency linked (PROJ-1102,
  open); no request volume given.
- **Flags:** none.
- **Route:** refinement.
- **Suggested rewrite:** none — no score at or below 2.
```

The empty and the unscoreable cases:

```
## PROJ-1201 — "Investigate"  (Task)

## Result
- **Clarity:** 1 — title only; description empty.
- **Scope:** 1 — nothing bounded.
- **Effort signal:** 1 — nothing to estimate from.
- **Flags:** description empty; no labels; no links.
- **Route:** back to reporter.

## Suggested rewrite
Not possible. The title "Investigate" carries no subject, so any rewrite would be
invention rather than tightening. Ask the reporter one question: what did you see,
and where?
```

```
## PROJ-1177 — "Handle the retry case"  (Subtask)

## Not scored
Subtasks are scored against their parent's goal, and the parent (PROJ-1150) was not
supplied. Scoring this in isolation would produce three numbers about the wrong thing.
Supply the parent and re-run.
```

## Edge cases
- **Bug tickets:** Clarity means reproduction steps plus expected versus actual. Missing repro caps Clarity at 2 [house rule].
- **Spikes and research tickets:** Scope is judged by the question, not by deliverables; a spike without a timebox caps Scope at 3 [house rule].
- **Empty description with a good title:** score it, do not refuse. The rewrite is the deliverable.
- **Empty description and an empty title.** Nothing to score against. Emit the "Not scored" shape and say the ticket carries no subject.
- **No ticket supplied**, or the paste is a Jira URL rather than the ticket text. Do not score a URL. Ask for the title, description and issue type.
- **Subtask with no parent supplied.** Do not score. Emit the "Not scored" shape above. Scoring a subtask in isolation reliably produces a Scope of 5 for work that is a fragment.
- **An epic pasted as a ticket**, or a description over ~1,500 words with more than five distinct requirements. Do not score it as one ticket. Report it as an epic, list the deliverables you can separate, and say each needs its own score.
- **Comments contradict the description.** The newest comment is the current truth. Score against it, and raise a flag naming both, because a ticket whose truth lives in comment 14 is a Clarity finding in itself.
- **The repro exists only in an attachment or a screenshot.** You cannot read it. Treat the repro as missing for scoring, and flag it as "repro may be in <filename>, not readable here" so a human can overturn the cap.
- **A ticket in a language you were not given a glossary for**, or one that is mostly internal shorthand. Score Clarity on structure, not vocabulary, and flag every term you could not resolve by name rather than guessing at it.
- **Acceptance criteria present but empty**, or a template left unfilled ("As a <user> I want <goal>"). That is worse than absent: score Clarity at 1 and flag the unfilled template explicitly, because these pass a template check.
- **Story points already set and high, on a ticket scoring 5 for Scope.** Mention the contradiction in one line. Do not change the score; points are not input to it.
- **A duplicate of a linked ticket.** Score it anyway, then flag the duplication with the linked key. Closing duplicates is not this skill's call.

## Stop and hand back
This skill routes tickets, and in the pipeline and scheduled modes it can write comments back to Jira where a reporter reads them. Stop, score nothing, and hand to a named person on any of these:

- **The ticket reports a security vulnerability, a data leak, or a suspected breach.** Do not score it, do not rewrite it, and do not post a comment asking for better repro steps — a public "please add reproduction steps" on a live vulnerability is itself a disclosure. Route to the security owner.
- **The description contains customer PII, credentials, tokens or payment data.** Stop and route to whoever owns that data. Say what was found and where; do not quote it in the output.
- **The ticket is a customer-facing escalation** — an SLA breach, a named account at risk, a complaint routed in from support. Score is the wrong tool. Hand to the support or account owner.
- **The ticket describes an active production incident.** Refinement scoring delays it. Hand to the on-call owner and say so in one line.
- **Any run that would write back to Jira** — posting a comment, changing a field, transitioning a ticket. Produce the output; do not write it. A human approves the write. In a scheduled sweep, publish the batch as a report and require sign-off before any comment is posted.
- **A sweep of more than 50 tickets.** Do not post 50 comments. Return the batch ranked worst-first with the counts, and let a person decide which reporters get a bounce. [house rule; the limit is about the reporters' inboxes, not the scoring.]
- **A ticket about a person's performance, conduct, or an HR matter** that has landed in the tracker. Do not score it. Route it out of the tracker and say only that.

## License
MIT
