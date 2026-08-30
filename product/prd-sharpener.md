---
name: prd-sharpener
owner: deckhand
category: Product
description: Turns a draft PRD into one a team can build from — problem before solution, success metric with a number, explicit non-goals, and open questions with owners.
version: v3
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/deckhand/prd-sharpener
raw: https://emdly.com/raw/deckhand/prd-sharpener.md
install: npx @emdly/cli add deckhand/prd-sharpener
---

# PRD sharpener

A PRD is done when an engineer can say what *not* to build. This skill edits a draft toward that: solution language out of the problem section, one success metric with a number on both ends, non-goals written down instead of assumed, and every open question owned by a person with a date. It sharpens what the author wrote. It does not add features, and it does not invent the evidence the author left out.

## When to use
- On any product doc before design or engineering review.
- On a feature request that arrived as a Slack message or a ticket comment and needs to become a document. This input is thinner than the structure below assumes; the branch for it is in "When the input is below draft strength".
- On a PRD that was reviewed once and came back as "what's the metric?".
- On a doc that has grown past two pages without a non-goals section.

## Input
Three things:
1. **The draft**, in any structure — Markdown, a doc export, a pasted Slack thread.
2. **The metric the team already tracks** that this work should move, with its current value and where it is read (dashboard, tag, query). "We'll add a metric" is not this.
3. **The constraints**: deadline, platforms, dependencies, and anything already decided.

If 2 or 3 are missing, the skill still runs but the Success section becomes a gap with a question attached, not a guess. See "Edge cases".

## The sharpened structure
Rewrite the draft into these seven sections, in this order.

1. **Problem** — who, what they are trying to do, what goes wrong today, and evidence. Evidence is one of a closed list: a quoted user sentence, a number with its source, a support ticket count with its tag or query, or a named incident. No solution words in this section.
2. **Success** — one primary metric with a current value, a target, and the date it will be read. At most two secondary metrics. [house rule] At least one guardrail metric: what must not get worse.
3. **Solution** — the smallest thing that could move the primary metric, described as behaviour the user can observe. Two alternatives considered, each with one sentence on why not.
4. **Non-goals** — an explicit list of what this does not do, weighted toward the things a reader would otherwise assume it does.
5. **Scope in stages** — only if it does not fit one release. Each stage says what it proves, not what it contains.
6. **Open questions** — each with an owner (a person, not a team) and a date. A question with no owner is a risk and is labelled one in the table.
7. **Risks** — what would make this fail even if built exactly as written.

## Rules
- Move solution language out of the Problem section, and quote each moved sentence back to the author under the rewritten paragraph, prefixed `> moved to Solution:`. Never delete a sentence silently.
- A primary metric with no current value is **flagged**: keep it, write `current: (not stated)`, and add an open question with an owner. A primary metric with no target is **rejected**: write `REJECTED — no target` and say what a target would look like. "Increase engagement" is not a metric and is rejected on both counts.
- Sanity-check the size of the ask. Divide the target delta by the time to the read date and say whether the described solution can plausibly move the metric that far. If it cannot, say so in one sentence with the arithmetic.
- Do not add features. Propose cuts, not additions. A cut is proposed as a non-goal, never applied silently.
- Keep the author's voice, terminology and product names. If they call it a "workspace", it stays a workspace.
- Never invent evidence, a number, a date, an owner or a decision. Anything you could not find in the input is written `(not stated)`.
- Output stays under 500 words excluding the tables and the moved-sentence quotes. A PRD nobody rereads is not sharp.

Thresholds above are defaults; report the thresholds you used.

## When the input is below draft strength
A Slack message is not a draft. Treat the input as below draft strength when fewer than three of the seven sections have any content in the input, or the whole input is under 200 words. [house rule]

Do not fill the other four sections. Emit the skeleton with named gaps and a question list addressed to the requester, in the order the answers unblock things: problem evidence first, metric second, constraints third. Write the sections you *can* fill from the input, and mark the rest `(not stated)`. Say plainly at the top that this is a stub, and what it needs before it is reviewable. A stub that admits it is a stub is useful; a stub padded into a full PRD is a fabrication with headings.

## Output format
Full draft in, sharpened PRD out.

```
## Problem (rewritten)
Shop owners who import product feeds cannot see which rows were dropped. They
discover it when a customer asks. 34 tickets tagged `feed-missing` in Q2
(Zendesk, 2026-04-01 to 2026-06-30) ask a version of "why is product X missing".
> moved to Solution: "we should add a validation report page"
> moved to Solution: "probably a nightly digest email too"

## Success
Primary: tickets/week tagged `feed-missing` — 2.6 now (34 over the 13 weeks of
Q2) → ≤ 1.0, read 2026-11-30.
Secondary: share of imports where the owner opens the report — (not stated), no
target proposed. Owner: PM.
Secondary: REJECTED — "increase engagement" is not a metric. A target would be
a named event, a current rate, and a date.
Guardrail: import time p95 stays under 90 s.
Plausibility: 2.6 → 1.0 is a 62% reduction in 13 weeks. The draft attributes
most of those tickets to silent drops, so the mechanism is there. Reachable.

## Solution
Every import writes a row-level result the owner can open from the import list:
accepted, dropped, and the reason per dropped row.
Considered: fix rows automatically — changes the owner's data without consent.
Considered: email the failures — 41% of these accounts have no verified address
(source: draft, unsourced; see open questions).

## Non-goals
- Fixing dropped rows automatically
- Supporting Shopify-format feeds
- Reports for imports that ran before launch
- Any change to the import scheduler

## Open questions
| question | owner | by |
|---|---|---|
| Keep dropped rows 30 or 90 days? | Priya (data) | 2026-09-05 |
| Where does the 41% unverified figure come from? | Sam (PM) | 2026-09-05 |
| Is legal OK with storing raw feeds? | **no owner — logged as a risk** | — |

## Risks
- Storing raw feed rows may hold customer PII with no retention rule agreed.
- If drops are mostly one malformed column, the report is right and useless.
```

Slack message in, stub out.

```
## Stub — not reviewable yet
Source: one Slack message, 61 words. Three of seven sections are empty. This is
what the message supports, and nothing more.

## Problem (from the message)
"Merchants keep asking why products vanish from their feed and we have no
answer." Who: merchants on the import path. What goes wrong: no answer exists.
Evidence: (not stated) — no count, no ticket tag, no quote from a merchant.

## Success
Primary: (not stated). REJECTED — no metric and no target were supplied, so
there is nothing to sharpen.

## Solution
(not stated) — the message asks for "some kind of visibility".

## Non-goals
(not stated)

## Open questions
| question | owner | by |
|---|---|---|
| How many tickets, over what window, under what tag? | (not stated) | — |
| Which tracked metric should this move, and its value today? | (not stated) | — |
| Deadline, platforms, dependencies? | (not stated) | — |

## Needed before this is reviewable
1. Ticket evidence: a count, a window, a tag or query.
2. One tracked metric with today's value and where it is read.
3. Constraints.
```

## Edge cases
- **No draft supplied.** Nothing to sharpen. Say so and stop. Do not write a PRD from the title.
- **Draft is below 200 words or fills fewer than three sections.** Use the stub branch above.
- **No metric supplied.** Success becomes `REJECTED — no target`, with an open question owned by the PM. Do not pick a metric for the team; choosing what a feature is for is their decision, and a sharpener that guesses it has written a different document.
- **No constraints supplied.** Sections 1-4 and 6-7 still work. Omit section 5 entirely and add one open question: "What is the deadline and what must this ship on?"
- **Metric supplied but not tracked anywhere.** Keep it, mark it `current: (not stated) — no source given`, and put the instrumentation in Open questions. An untracked metric is a project, and it belongs in the doc as one.
- **Draft over 4,000 words or more than one feature.** Sharpen the first feature only, in document order, and say which sections and which feature you covered and what you left. Do not compress two features into one PRD.
- **Malformed input** (a doc export with tracked-changes markup, a screenshot transcript, HTML nav and footer). Strip navigation, comments and revision marks; if more than a quarter of the text is unreadable, say the input is unusable and ask for a plain-text paste rather than guessing at the prose.
- **The draft's evidence contradicts itself** (two different ticket counts, two different dates). Quote both, do not pick, and file it as an open question with the PM as owner.
- **The author already wrote non-goals.** Keep every one of them verbatim, then add what you found missing beneath. Deleting a non-goal is a scope change, which is not this skill's call.

## License
MIT
