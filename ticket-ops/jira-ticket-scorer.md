---
name: jira-ticket-scorer
owner: opsmith
category: Ticket ops
description: Scores every Jira ticket for clarity, scope and effort before it hits the sprint. Flags vague acceptance criteria and suggests a rewrite.
version: v3
license: MIT
updated: 2026-08-25
recommended: true
security_checked: true
url: https://emdly.com/skills/opsmith/jira-ticket-scorer
raw: https://emdly.com/raw/opsmith/jira-ticket-scorer.md
install: npx @emdly/cli add opsmith/jira-ticket-scorer
---

# Jira ticket scorer

Score a ticket **before** it is estimated, so the team argues about the work, not about what the words mean.

## When to use
- As a step in a pipeline: new ticket → score → route to refinement or back to the reporter.
- Standalone in Claude Code: paste the ticket, get the score and the rewrite.
- On a schedule via the API, sweeping everything created since the last run.

## Input
The ticket's title, description, acceptance criteria (if any), labels and linked issues. Comments are optional; when present, treat the newest comment as the current truth.

## Rubric
Score three dimensions from 1 to 5. Do not average them — report each.

**Clarity** — Could an engineer who has never seen this codebase start without asking a question?
- 5: user, trigger, expected outcome and out-of-scope are all explicit.
- 3: the goal is clear, the boundary is not.
- 1: a title and a sentence.

**Scope** — Is this one deliverable?
- 5: one change, one owner, one way to verify.
- 3: two or three related changes that could ship separately.
- 1: a project wearing a ticket's clothes.

**Effort signal** — Does the ticket contain what estimation needs (affected components, data, dependencies)?
- 5: components named, edge cases listed, dependencies linked.
- 1: nothing to estimate from.

## Rules
- Never invent acceptance criteria that are not in the ticket. Propose them, and label the proposal.
- Flag anything you could not verify instead of guessing. "The reporter probably means…" is a flag, not a fact.
- Subtasks inherit their parent's context: score them against the parent's goal, not in isolation.
- Story points already on the ticket are not evidence of clarity. Ignore them for scoring; mention them only if they contradict the effort signal.
- Keep the rewrite in the reporter's voice. You are tightening, not redesigning the feature.

## Output format
```
## Result
- **Clarity:** 3 — goal clear, boundary missing (what happens for guest users?)
- **Scope:** 2 — contains a migration and a UI change that can ship separately
- **Effort signal:** 4 — components named, no data volume given
- **Flags:** "as before" refers to behavior not described anywhere; linked PROJ-812 is closed
- **Suggested rewrite:** only when any score is ≤ 2
```
Keep the whole result under 200 words. One ticket, one result.

## Edge cases
- Bug tickets: Clarity means reproduction steps + expected vs. actual. Missing repro caps Clarity at 2.
- Spikes and research tickets: Scope is judged by the question, not by deliverables; a spike without a timebox caps Scope at 3.
- Empty description with a good title: score it, do not refuse. The rewrite is the deliverable.

## License
MIT
