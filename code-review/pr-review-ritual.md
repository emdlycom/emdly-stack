---
name: pr-review-ritual
owner: kernelpanic
category: Code review
description: A reviewing discipline for agents — read the diff twice, test the edge cases, comment on intent, never nitpick formatting a linter owns.
version: v3
license: MIT
updated: 2026-08-20
recommended: true
security_checked: true
url: https://emdly.com/skills/kernelpanic/pr-review-ritual
raw: https://emdly.com/raw/kernelpanic/pr-review-ritual.md
install: npx @emdly/cli add kernelpanic/pr-review-ritual
---

# PR review ritual

A review is a claim: "I understand what this change does, and here is what it risks." This skill makes an agent earn that claim before it writes a single comment.

## When to use
- On every pull request, as a first pass before a human reviewer.
- In Claude Code on a local branch before opening the PR.

## Input
The diff, the PR description, the linked issue if any, and — when available — the CI result and the files' git history.

## The ritual
1. **Read the description, then the diff, then the description again.** Note every place where they disagree. A diff that does more than the description says is the most common finding.
2. **Second read, for intent.** For each hunk, say in one sentence why it exists. If you cannot, that hunk needs a comment or a question — not a guess.
3. **What would break.** For every changed function, list the callers you can see and the inputs that changed meaning: null, empty, huge, concurrent, retried. Trace at least one path end to end.
4. **Tests.** Does a test fail if this change is reverted? If no test covers the new behavior, say which case is missing — specifically, with the input.
5. **Write comments.** Each comment quotes the line, states the risk, and, when you have one, offers the fix. Questions are allowed and are better than wrong assertions.

## Severity ladder
- **Blocker** — data loss, security, a broken invariant, a public API change without a note.
- **Should fix** — a bug on a reachable path, a missing test for new behavior, an unclear name that will mislead the next reader.
- **Consider** — a simpler way, a duplicated helper, a comment that no longer matches the code.
- **Nit** — do not write these. Formatting, import order and whitespace belong to the linter; if the repo has no linter, say that once at the top, not per line.

## Rules
- Never approve. You are the first reader, not the owner. End with a summary and the list of blockers.
- Never rewrite the PR in a comment. Show the smallest change that resolves the point.
- Quote, don't paraphrase — a comment the author cannot find is noise.
- If the diff is over ~800 lines, say so first and review the riskiest files, named.

## Output format
```
## Summary
Adds retry with backoff to the webhook sender. The description says "retries 3×"; the code retries 5× (sender.php:41).

## Blockers (1)
- sender.php:58 — `sleep()` inside the request cycle; a slow webhook now blocks the worker for up to 31 s.

## Should fix (2)
- ...

## Missing tests
- retry gives up after the last attempt and records the failure (no test asserts the failure path)
```

## License
MIT
