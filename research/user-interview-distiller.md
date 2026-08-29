---
name: user-interview-distiller
owner: fieldnotes
category: Research
description: Distills interview transcripts into claims with evidence — what was said, how often, and what it contradicts in your assumptions.
version: v3
license: MIT
updated: 2026-07-31
recommended: false
security_checked: true
url: https://emdly.com/skills/fieldnotes/user-interview-distiller
raw: https://emdly.com/raw/fieldnotes/user-interview-distiller.md
install: npx @emdly/cli add fieldnotes/user-interview-distiller
---

# User interview distiller

Interviews produce stories; decisions need claims. This skill turns transcripts into claims that carry their evidence, so nobody has to trust a summary.

## When to use
- After a round of 5–12 interviews, on the transcripts.
- Before a roadmap meeting where "users want X" is about to be said.

## Input
Transcripts (with participant ids), the research questions, and the team's assumptions going in (a list — even a rough one).

## Process
1. **Code first, count second.** Read each transcript and tag statements with a short code (e.g. `export-manual`, `trust-numbers`). Codes come from what people say, not from the assumptions.
2. **Claims.** For each code seen in ≥ 2 participants, write a claim in neutral language and attach: participant count, and the two most representative quotes with ids.
3. **Single-voice items** are listed separately — real, but not a pattern yet.
4. **Assumptions cross-check.** For each assumption: supported / contradicted / not addressed, with the claim that decides it.
5. **Surprises.** Things nobody expected and no question asked about — quoted.

## Rules
- Every claim quotes. A claim without a quote is deleted.
- Count participants, not mentions.
- Never turn a request into a claim about need: "P3 asked for CSV export" is evidence for "P3 wants to work with the data elsewhere", and the claim says the latter with the quote.
- Leading questions in the transcript are flagged, and answers to them are down-weighted.
- Preserve the participants' words; do not clean up their grammar.

## Output format
```
## Claims (8 participants)
**C1 · 6/8 — They rebuild our numbers in a spreadsheet before trusting them.**
- P2: "I export it, sum it myself, and only then do I put it in the deck."
- P7: "the dashboard says one thing, finance says another, so I check"
→ contradicts assumption A3 ("users trust the dashboard totals").

**C2 · 4/8 — Onboarding is done by a colleague, not by the docs.** …

## Single voices
- P5 wants a Slack digest (no one else mentioned notifications).

## Assumptions
| A1 weekly usage | supported (C4) | A3 trust totals | contradicted (C1) | A5 mobile use | not addressed |

## Surprises
- Three participants use the product for a purpose we never mention: … (P1, P4, P8)
```

## License
MIT
