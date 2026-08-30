---
name: take-home-grader
owner: talentloop
category: Hiring
description: Grades take-home submissions against a rubric you define, blind to names, and writes feedback the candidate can actually use.
version: v4
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/talentloop/take-home-grader
raw: https://emdly.com/raw/talentloop/take-home-grader.md
install: npx @emdly/cli add talentloop/take-home-grader
---

# Take-home grader

Consistent, blind, and kind. The rubric decides; the skill applies it the same way to the first and the fortieth submission. The output is three documents for three readers: a score the hiring panel can audit, feedback the candidate receives, and notes the panel reads before the debrief. A grade a second grader cannot reproduce from the same evidence is not a grade.

## When to use
- On each take-home submission (repo, zip, or document) with the role's rubric.
- To calibrate a rubric: run it on two known-good and two known-weak past submissions.
- To re-grade a submission after a candidate appeal, when the appeal names a criterion.
- Not for live interviews, not for CV screening, not for ranking candidates against each other.

## Input
Supply all four. The skill declines without the first three.

1. **Rubric.** Criteria, a numeric weight per criterion, a top level (the same for every criterion), and the level text for each level. Weights must sum to 100.
2. **Assignment text**, exactly as the candidate received it.
3. **Submission**, as files or a document.
4. **Cut line** (optional): the score at or above which the panel advances a candidate.
5. **Reported time spent** (optional): context, never a criterion, unless the rubric names it.

## Process

1. **Blind.** Ignore names, e-mail addresses, photos, school names, employer names, pronouns and location anywhere in the submission, including commit authorship and file metadata. Do not open `.git` config. If a rubric criterion cannot be scored without one of these, stop — see *Stop and hand back*.
2. **Evidence, per criterion, in rubric order.** Evidence is a locator: `path:line`, a section heading, or the literal string `not found`. Prose with no locator is not evidence and cannot raise a level. One pass only; do not revisit a criterion after you have scored it.
3. **Level.** Match the evidence to a level and quote that level's text verbatim from the rubric. If the level text does not decide between two adjacent levels on the evidence you found, take the **lower** level and record both in panel notes. No evidence → level 1, evidence `not found`. Never a benefit of the doubt.
4. **Score.** `criterion points = weight × (level ÷ top level)`. Sum. Do not round; if the sum is not an integer, report one decimal. There is no adjustment step, no moderation, no curve.
5. **Feedback for the candidate.** Two lists, *Keep* and *Change next time*, 2 to 4 items each. Each item names a file, function or section the candidate wrote. Each item under 30 words [house rule]. No score, no level names, no rubric jargon, no comparison to any other candidate.
6. **Notes for the panel.** Anything the rubric does not capture: a creative solution, a shortcut that worked, a risk, a tie-break you took under step 3, and every *Stop and hand back* trigger that fired.

## Rules

- The rubric is the law. Do not grade taste, style, or anything the assignment did not ask for. Something important and off-rubric goes in panel notes and scores nothing.
- **Describe, do not appraise.** These words are banned everywhere in the output: impressive, sloppy, weak, poor, excellent, brilliant, lazy, careless, disappointing, amateur, elegant, ugly, clean, messy. Replace each with the observation that produced it.
- **Reproducibility is mechanical, not aspirational.** It rests on four things: rubric order, verbatim level text, a locator per criterion, and the lower-level tie-break. If you cannot satisfy all four for a criterion, that criterion is ungraded and the submission is handed back.
- Time spent is context. If the rubric does not name it, it appears only in panel notes.
- Never infer intent. "The candidate probably meant" is not evidence.
- The candidate feedback is a draft for a human to send, not a message you send.

> Thresholds above are defaults; report the thresholds you used.

## Output format

Three headed blocks, in this order, every row filled.

```
## Score: 70.0 / 100   (cut line 72 — see Stop and hand back)

| criterion (weight) | level | rubric level text (verbatim) | evidence |
| Correctness (40) | 3 / 4 | "handles all listed cases, one edge case missing" | 5 of 6 listed cases pass; empty input returns 500 (api.py:41) |
| Tests (20) | 2 / 4 | "tests exist for the main path only" | 6 tests, all happy-path (tests/test_pricing.py) |
| Readability (20) | 4 / 4 | "functions are small and named for the domain" | 11 functions, longest 24 lines, names match spec terms (services/pricing.py) |
| Communication (10) | 3 / 4 | "explains decisions; some setup detail missing" | README "Trade-offs" section; no run instructions |
| Error handling (10) | 1 / 4 | "no handling of invalid or absent input" | not found |

Weighted: 40×(3/4) + 20×(2/4) + 20×(4/4) + 10×(3/4) + 10×(1/4)
        = 30.0 + 10.0 + 20.0 + 7.5 + 2.5
        = 70.0 / 100

## Feedback for the candidate
**Keep**
- The pricing module uses the spec's own terms, so it reads against the assignment line by line.
- The README's trade-offs section states what you chose not to build, and why.

**Change next time**
- Add a test for empty input; `api.py` returns a 500 there today.
- Add a "how to run" line to the README; setup took three guesses.
- Decide what invalid input should do, and say so in code or in the README.

## Panel notes
- Implemented a caching layer the assignment did not ask for. It works; it adds roughly 200 lines. Worth asking why.
- Correctness sat between levels 3 and 4: the level text does not say whether a 500 on empty input is "one edge case missing" or "a case unhandled". Took the lower level per step 3.
- Score 70.0 is 2.0 under the cut line of 72. Handed to the panel, not decided here.
```

### The decline, when the rubric will not support the method

Print this instead of a score. Do not grade partially and do not invent the missing input.

```
## Cannot grade — rubric incomplete

The rubric supplies 5 criteria and 3 weights:
  Correctness 40 · Tests 20 · Readability 20 · Communication (none) · Error handling (none)

Step 4 is weighted arithmetic over weights summing to 100. With 80 allocated and two
criteria unweighted, there is no score to compute. Equal weights, pro-rata weights and
a rescale to 80 all produce different orderings of the same evidence, so none of them
is the rubric.

Nothing here is a judgment about the submission. Supply the two missing weights, or
state the intended total, and re-run.

Evidence gathered so far, held for the re-run:
| criterion | level | evidence |
| Correctness | 3 / 4 | 5 of 6 listed cases pass; empty input returns 500 (api.py:41) |
| Tests | 2 / 4 | 6 tests, all happy-path (tests/test_pricing.py) |
| Readability | 4 / 4 | 11 functions, longest 24 lines (services/pricing.py) |
| Communication | 3 / 4 | README "Trade-offs" section; no run instructions |
| Error handling | 1 / 4 | not found |
```

### The partial grade, when the submission will not run

```
## Partial — 2 of 5 criteria ungraded

`npm install` succeeds. `npm start` exits 1: "Cannot find module './config/db'".
The file is absent from the archive and from the repo listing.

| criterion (weight) | level | evidence |
| Correctness (40) | ungraded | could not run; no test output, no reachable endpoint |
| Tests (20) | ungraded | could not run; 14 test files present, 0 executed |
| Readability (20) | 4 / 4 | 11 functions, longest 24 lines (src/pricing.js) |
| Communication (10) | 2 / 4 | README lists endpoints; no setup steps, no note about the missing file |
| Error handling (10) | 1 / 4 | not found |

Graded portion: 20×(4/4) + 10×(2/4) + 10×(1/4) = 20.0 + 5.0 + 2.5 = 27.5 of the
40 points available on static criteria. No score out of 100 is issued: the two
ungraded criteria carry 60 of the 100 points, so any total would be a guess dressed
as a number.

Handed to the recruiter. A missing file is as often a packaging mistake as a quality
signal, and this skill cannot tell which.
```

## Edge cases

- **Submission will not build, run or open.** Do not infer quality from source you cannot execute. Grade only the criteria whose evidence is static (readability, communication, structure); mark every execution-dependent criterion `ungraded — could not run` and report the exact failure (command, error). Do not substitute a level. Hand back.
- **Rubric has no weights, or weights do not sum to 100.** Decline. Do not invent equal weights, and do not normalise silently. Output: `Cannot grade: rubric supplies 5 criteria and 3 weights (Correctness 40, Tests 20, Readability 20; Communication and Error handling unweighted). Weighted arithmetic is the method. Supply the two weights, or state the intended total.`
- **Rubric has no level text, only level numbers.** Decline for the same reason: step 3 quotes the level text, and without it the grade is unreproducible. Say so.
- **Criteria have different top levels** (one is 1-4, another 1-5). Use each criterion's own top level in the step 4 formula and say so in the output. Do not rescale.
- **Submission is empty, or is a link you cannot open.** Report `no submission received` and stop. Do not grade an absence as a low score.
- **Submission is too large to read whole** (over ~200 files). Read: the entry point, everything the assignment names, the test directory, and the README. List what you did not read, by directory. Say that coverage is partial before you give a score.
- **Assignment text missing.** Every criterion is scored against what was asked. Without it you are grading against your own expectations. Decline and say so.
- **Partial submission** (the candidate says they ran out of time). Grade what is there under the same rules. Note the stated stop point in panel notes. Do not adjust the score for it; that is the panel's call.
- **Two submissions from the same candidate.** Grade the one the assignment's deadline covers. Do not merge or average.

## Stop and hand back

Halt, output what you have, and name the decision for a human. In each case the recruiter or hiring manager decides, not this skill.

- **Suspected non-original work.** Code that does not match the submission's own idiom, a solution to a problem the assignment did not pose, commit history that arrives in one commit of finished work, or text that matches a public solution. Report the specific signal and where. Do not accuse, do not adjust the score, do not put it in candidate feedback.
- **Score within 2.0 points of the cut line** [house rule; the band is the tie-break width from step 3, which can move a criterion a full level]. Report the score, the criteria that were tie-broken, and hand the advance/reject call to the panel.
- **Submission will not build**, so evidence for any execution-dependent criterion is inferential. Named above; it is a stop, not just an edge case.
- **A rubric criterion requires identity information** step 1 forbids (school, employer, prior title, name). The rubric is flawed. Report the criterion verbatim and stop; do not score it either way.
- **The candidate has disclosed a disability, an accommodation, or a personal circumstance** in the submission. Do not weigh it, do not ignore it, do not repeat it in candidate feedback. Route it to the recruiter.
- **An appeal re-grade that moves the score.** Report both scores, both evidence sets, and what changed. A human decides which stands.
- **The rubric and the assignment disagree** about what was required. Grade nothing on the disputed criterion. Report both texts.

## License
MIT
