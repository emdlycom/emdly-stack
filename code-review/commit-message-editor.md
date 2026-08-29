---
name: commit-message-editor
owner: kernelpanic
category: Code review
description: Rewrites a commit message from the diff — imperative subject under 72 characters, a body that explains why, and a footer that links the issue.
version: v2
license: MIT
updated: 2026-08-02
recommended: false
security_checked: true
url: https://emdly.com/skills/kernelpanic/commit-message-editor
raw: https://emdly.com/raw/kernelpanic/commit-message-editor.md
install: npx @emdly/cli add kernelpanic/commit-message-editor
---

# Commit message editor

Good history is written for the person doing `git blame` at 2 a.m. This skill writes messages for that person.

## When to use
- Before committing, in Claude Code: `git diff --staged` in, message out.
- In a pre-commit hook that rejects messages under a quality bar.

## Input
The staged diff, the branch name, the last 20 subjects of the repo (`git log --oneline -20`), and the issue reference if the branch name carries one.

## Rules
- **Match the house style first.** If the last 20 subjects use Conventional Commits (`feat:`, `fix:`), use it. If they use ticket prefixes (`PROJ-12:`), use that. Never introduce a convention the repo does not have.
- Subject: imperative mood, ≤ 72 characters, no trailing period, says what the change does to the system — not what you did ("Add retry to webhook sender", not "Added retries").
- Body: **why**, not what. The diff already shows what. Explain the constraint, the bug, or the decision that a reader could not infer.
- One logical change per commit. If the diff needs "and" to be described, propose a split with the file groups.
- Footer: `Refs PROJ-12` / `Closes #42` only when the reference exists in the input.
- Never mention the tool that wrote the message.

## Output format
```
Add exponential backoff to webhook sender

Webhook targets that return 5xx were retried immediately three times,
which turned a receiver's brief outage into a burst of duplicate calls.
Backoff starts at 1 s and caps at 30 s; the final failure is recorded
so the dashboard can show it.

Refs PROJ-418
```
If a split is needed, return one message per proposed commit with the files listed under each.

## Edge cases
- Merge and revert commits keep git's default subject; only add a body when the reason is not obvious.
- A diff of only generated files gets a one-line subject and no body.

## License
MIT
