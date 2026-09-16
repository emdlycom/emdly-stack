---
name: sprint-goal-summary
owner: launifycorp
category: Team ops
description: Turn raw sprint planning notes into a single, testable sprint goal plus a scoped commitment summary the team and stakeholders can act on. You own the output that answers three questions: what the spri...
version: v1
license: MIT
updated: 2026-09-16
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/sprint-goal-summary
raw: https://emdly.com/raw/launifycorp/sprint-goal-summary.md
install: npx @emdly/cli add launifycorp/sprint-goal-summary
---

# Sprint Goal Summary

Turn raw sprint planning notes into a single, testable sprint goal plus a scoped commitment summary the team and stakeholders can act on. You own the output that answers three questions: what the sprint is trying to achieve, what is in and out of scope, and how anyone will know at sprint end whether the goal was met.

## When to use

- Sprint planning just ended and you have notes, a ticket list, or a transcript that nobody has turned into a stated goal.
- The backlog items for the sprint are selected but the team cannot articulate a single unifying objective.
- A stakeholder asks "what is this sprint about?" and the only answer available is a list of ticket IDs.
- Mid-sprint scope changed and the existing goal statement no longer matches what the team is building.
- You need a sprint summary for a status report, steering update, or release note draft.

Do not use when:

- Sprint content is still being negotiated and items are not yet selected — the goal will churn; wait for commitment.
- The request is a retrospective, velocity analysis, or sprint review of completed work; those need outcome data, not planning notes.

## Inputs

Required before starting:

- Sprint planning notes, transcript, or meeting summary.
- The list of committed backlog items: ID, title, and ideally estimate.
- Sprint dates (start and end) and sprint number or name.
- Team name and product/component area.

Strongly preferred:

- Known dependencies, blockers, or external commitments.
- Team capacity for the sprint (person-days, story points, or headcount with time off).
- The product goal, roadmap theme, or OKR this sprint rolls up to.
- Items explicitly discussed and deliberately excluded.

If any required input is missing, ask for it once, in a single consolidated question, and name the specific artifact you need. If the requester cannot supply it:

- Missing sprint dates or number: proceed and mark those fields `[TBD]`.
- Missing committed item list: stop. You cannot scope a sprint without knowing the work. Ask for the board export, ticket list, or screenshot.
- Missing capacity: proceed, omit the capacity line, and note the omission in Open Questions.
- Missing product goal: proceed, leave the "Rolls up to" line as `[not stated in notes]`.

## Method

1. Read the full input once without writing anything. If the notes cover more than one sprint or more than one team, ask which one to summarize rather than merging them.
2. Extract every committed work item into a list with ID, title, and estimate. If an item is mentioned but its commitment status is ambiguous ("we might get to the export thing"), place it in Stretch, not Committed.
3. Cluster the committed items by the user-facing or system-facing outcome they produce, not by component or assignee. If one cluster holds 60% or more of the estimated effort, that cluster is the goal candidate. If no cluster reaches 60%, pick the cluster with the highest stated business or stakeholder priority and label the remainder as supporting work.
4. Draft the sprint goal as one sentence in the form: enable/deliver/reduce `<outcome>` for `<who>` so that `<why it matters>`. Reject any draft that names a ticket ID, a component, or a sprint number; those describe work, not a goal.
5. Test the goal against a demo check: can the team show this at sprint review in under five minutes? If not, narrow it. If the goal restates the entire backlog list, it is not a goal — narrow it to the dominant outcome.
6. Write 2–4 success criteria. Each must be verifiable at sprint end by observation or a number, phrased as a condition that is true or false. Reject criteria containing "improve", "better", or "more" without a threshold.
7. Assign every committed item to In Scope. Move anything conditional to Stretch. Build the Out of Scope list only from items explicitly discussed and rejected in the notes — never invent exclusions to look thorough.
8. Extract dependencies and risks. For each, record the owner and the date or condition by which it must resolve. If the notes name a risk with no owner, record the risk and put the owner as `[unassigned]` rather than guessing.
9. Compute the load line: total committed estimate against capacity if both are available. If committed effort exceeds stated capacity, flag it in Risks with the exact numbers; do not silently soften it.
10. List Open Questions for anything you had to infer, any unassigned owner, and any item whose scope was unclear in the notes. If Open Questions would exceed five entries, that signals the planning notes are too thin — say so at the top of the output.
11. Re-read the final summary against the source notes and remove any claim you cannot trace to a line in the input.

## Rules

- Never invent work items, dates, estimates, owners, or acceptance criteria. Every fact in the output must be traceable to the input.
- One sprint goal only. If the notes genuinely contain two unrelated objectives, state one goal and list the second as a secondary objective in Scope, then flag the split in Risks.
- The goal sentence is one sentence, maximum 30 words, and contains no ticket IDs, component names, or internal jargon a new stakeholder could not parse.
- Never convert an unstated assumption into a commitment. Ambiguity goes to Open Questions, not to In Scope.
- Do not restate the ticket list as the goal. If the goal and the In Scope list are the same content, redo step 3.
- Success criteria must be checkable on the sprint end date. Anything measurable only weeks later belongs in Notes, not criteria.
- Keep the whole deliverable under one page. Cut detail from Scope tables before cutting the goal, criteria, or risks.
- Do not editorialize on team performance, capacity decisions, or stakeholder behavior. Report the load line as numbers and stop.
- Preserve the exact ticket IDs and titles from the source; do not paraphrase item titles.
- Mark every inferred value inline with `[inferred]` so a reader can challenge it.

## Output format

```
# Sprint Goal Summary — [Team] — Sprint [N]
Dates: [start] – [end]
Rolls up to: [product goal / OKR / theme, or "not stated in notes"]

## Sprint Goal
[One sentence, max 30 words, outcome-framed.]

## Success Criteria
1. [Verifiable condition, true/false at sprint end]
2. [Verifiable condition]
3. [Verifiable condition]

## In Scope (committed)
| ID | Item | Estimate | Supports goal? |
|----|------|----------|----------------|
| [ID] | [title] | [pts/days] | Yes / Supporting |
| [ID] | [title] | [pts/days] | Yes / Supporting |

Total committed: [X] | Capacity: [Y] | Load: [X/Y]

## Stretch (only if capacity allows)
- [ID] — [title] — [estimate]

## Out of Scope (discussed and excluded)
- [item] — reason: [as stated in notes]

## Dependencies
- [dependency] — owner: [name] — needed by: [date/condition]

## Risks
- [risk] — impact: [what breaks] — mitigation/owner: [as stated, or "unassigned"]

## Open Questions
- [question] — needs answer from: [role/name] — by: [date]

## Notes
- [inferences made, source gaps, anything marked [inferred] above]
```

## Failure modes

- **Goal is a disguised task list.** The "goal" reads "finish checkout refactor, ship the export, fix the login bug." Check: cover the In Scope table and ask whether the goal alone tells a stakeholder why the sprint matters. If it does not, return to step 3 and recluster by outcome.
- **Fabricated scope or owners.** Items, estimates, or risk owners appear that the notes never mentioned, usually because the summary "felt incomplete." Check: line-by-line, point each output element at the source text. Anything with no source line gets deleted or moved to Open Questions.
- **Unverifiable success criteria.** Criteria read "checkout is faster" or "the team is unblocked." Check: for each criterion, name the exact observation or number you would look at on the last day of the sprint. If you cannot name it, rewrite with a threshold or drop it.
- **Overcommitment hidden by confident phrasing.** Committed effort exceeds capacity but the summary reads as a clean plan. Check: compute total committed against capacity before writing the goal. If load exceeds 100%, the Risks section must contain the numeric overage; if capacity is unknown, Open Questions must say so.

## License

MIT
