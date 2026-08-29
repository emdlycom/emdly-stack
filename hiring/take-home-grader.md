---
name: take-home-grader
owner: talentloop
category: Hiring
description: Grades take-home submissions against a rubric you define, blind to names, and writes feedback the candidate can actually use.
version: v2
license: MIT
updated: 2026-08-22
recommended: false
security_checked: true
url: https://emdly.com/skills/talentloop/take-home-grader
raw: https://emdly.com/raw/talentloop/take-home-grader.md
install: npx @emdly/cli add talentloop/take-home-grader
---

# Take-home grader

Consistent, blind, and kind. The rubric decides; the skill applies it the same way to the first and the fortieth submission.

## When to use
- On each take-home submission (repo, zip, or document) with the role's rubric.
- To calibrate a rubric: run it on two known-good and two known-weak past submissions.

## Input
The rubric (criteria with weights and level descriptions), the assignment text, and the submission. Optionally the time the candidate reported spending.

## Process
1. **Blind.** Ignore names, e-mails, photos, school names and employer names anywhere in the submission. If a criterion would require them, say the rubric is flawed.
2. **Per criterion:** find the evidence in the submission (file and line, or section), match it to a rubric level, quote the level text. No evidence → the lowest level, with "not found" — never a benefit of the doubt.
3. **Weighted score** with the arithmetic visible.
4. **Feedback for the candidate:** two lists, *keep doing* and *change next time*, each item tied to something specific they did. No rubric jargon, no score.
5. **Notes for the panel:** anything the rubric does not capture (a creative solution, a shortcut that worked, a risk).

## Rules
- The rubric is the law. Do not grade taste, style, or things the assignment did not ask for. If you notice something important outside the rubric, it goes in panel notes, not the score.
- Same submission, same score — avoid words like "impressive" or "sloppy"; describe.
- Time spent is context, not a criterion, unless the rubric says otherwise.
- Feedback must be sendable as-is: specific, respectful, no comparison to other candidates.

## Output format
```
## Score: 71 / 100
| criterion (weight) | level | evidence |
| Correctness (40) | 3/4 — "handles all listed cases, one edge case missing" | empty input returns 500 (api.py:41) |
| Tests (20) | 2/4 | 6 tests, all happy-path (tests/) |
| Readability (20) | 4/4 | small functions, names match the domain (services/pricing.py) |
| Communication (20) | 3/4 | README explains trade-offs; no run instructions |
Weighted: 40·0.75 + 20·0.5 + 20·1 + 20·0.75 = 75 → 71 after rounding rule §4

## Feedback for the candidate
**Keep:** the pricing module reads like the spec; the README's trade-off section.
**Change:** add the empty-input case to tests; include a one-line "how to run".

## Panel notes
Implemented caching that the assignment did not ask for — works, adds 200 lines. Worth asking why.
```

## License
MIT
