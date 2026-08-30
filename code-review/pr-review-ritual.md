---
name: pr-review-ritual
owner: kernelpanic
category: Code review
description: A reviewing discipline for agents — read the diff twice, test the edge cases, comment on intent, never nitpick formatting a linter owns.
version: v5
license: MIT
updated: 2026-08-30
recommended: true
security_checked: true
url: https://emdly.com/skills/kernelpanic/pr-review-ritual
raw: https://emdly.com/raw/kernelpanic/pr-review-ritual.md
install: npx @emdly/cli add kernelpanic/pr-review-ritual
---

# PR review ritual

A review is a claim: "I understand what this change does, and here is what it risks." This skill makes an agent earn that claim before it writes a single comment. The output is for the human who will approve or reject the PR: it must let them act without re-reading the diff, every point must name a file and a line, and it must say plainly which parts of the change the agent could not verify.

## When to use
- On every pull request, as a first pass before a human reviewer.
- In Claude Code on a local branch before opening the PR.
- On a stacked branch, reviewing the top commit against its parent only.
- After a force-push or rebase, when the previous review no longer matches the code.
- When CI is green and nobody can say why the change is safe.

## Input
Required. Without both of these the ritual does not run:
- The unified diff with file paths and line numbers (`git diff`, or the PR's `.diff`).
- The PR description, or an explicit note that there is none.

Optional. Each one widens what the review can prove, so name the ones you had:
- The linked issue or ticket.
- The CI result, per job, including which jobs were skipped.
- The repository at the base commit. Without it you cannot see callers, and step 3 degrades to a guess.
- `git log --follow` for the changed files.
- The repo's linter and formatter config, so you know what the linter already owns.

Open the review with a one-line `Inputs:` list naming what you had and what you did not. A review that could not see callers is a different review, and the reader must be told which one they are getting.

## The ritual
1. **Read the description, then the diff, then the description again.** Write down every place they disagree. A diff that does more than the description says is the most common finding in this catalog's experience, and it goes in the summary, not in a line comment.
2. **Second read, for intent.** For each hunk, write one sentence saying why it exists. If you cannot write that sentence, the hunk gets a question, not a guess. Count the hunks you could not explain and report the count.
3. **What would break.** For every changed function, list the callers you can see and the inputs whose meaning changed: null, empty, zero, negative, huge, unicode, concurrent, retried, replayed. Trace at least one path end to end, naming each file it passes through. If you have no repo access, say "callers not checked — no repository access" and do not assert impact.
4. **Tests.** For each behavior change, ask: does a test fail if this hunk is reverted? If no test covers the new behavior, name the missing case with the concrete input, not the concept. "No test for empty payload" is a finding; "test coverage could be better" is not.
5. **Write comments.** Each comment quotes the line verbatim, states the risk in one sentence, and offers the smallest fix when you have one. Questions are allowed and are better than wrong assertions.

## Severity ladder
Use exactly these four labels. Do not invent a fifth.

- **Blocker** — data loss, a security hole, a broken invariant, a public API or schema change with no migration or note, a secret committed to the repo.
- **Should fix** — a bug on a path a caller can reach, a missing test for new behavior, a name that will mislead the next reader.
- **Consider** — a simpler construction, a helper that already exists, a comment that no longer matches the code.
- **Nit** — do not write these. Formatting, import order and whitespace belong to the linter. If the repo has no linter config in the input, say so once in the summary and never per line.

## Rules
- Never rewrite the PR in a comment. Show the smallest change that resolves the point, and nothing else.
- Quote, do not paraphrase. A comment the author cannot locate by searching for the quoted text is noise.
- Every finding carries `path:line`. A finding without a location is deleted before you publish.
- Counts in the headings must equal the bullets under them. `## Blockers (1)` has exactly one bullet.
- Over **800 changed lines** (added plus removed, excluding lockfiles, generated files and vendored directories), do not claim a complete review. Say so in the first line, name the three riskiest files, and review only those. [house rule, anchored on the Cisco/SmartBear code review study, which found defect detection falls off past roughly 200-400 lines in one sitting; 800 is where this skill stops pretending.]
- More than **20 findings** means the review is unreadable. Publish the blockers, the top five Should fix, and one line: "N further Should fix / Consider items withheld; the change is too large for one review."
- Report every number you assert as a count, never as "several" or "many".

> Thresholds above are defaults; report the thresholds you used.

## Output format

```
Inputs: diff, description, CI (3 jobs, 1 skipped). No repository access — callers not checked.

## Summary
Adds retry with backoff to the webhook sender. The description says "retries 3x"; the code
retries 5x (sender.php:41). Two of eleven hunks I could not explain from the diff alone; both
are in the queue worker and are marked with questions below. The repo ships no linter config in
this input, so formatting is not reviewed.

## Blockers (1)
- sender.php:58 — `sleep($delay);` runs inside the request cycle. With the new 5 attempts and a
  30 s cap the worker is blocked for up to 31 s per webhook, so one slow receiver stalls the
  whole queue. Move the wait to the scheduler: re-queue with `available_at = now + $delay`.

## Should fix (2)
- sender.php:44 — `$delay = $this->policy->delays[$attempt];` with `$attempt` starting at 1
  and `RetryPolicy::$delays` holding five values reads `delays[1]` on the first wait and
  `delays[5]` on the fifth, one past the end. `delays[0]` is never used, and the last read
  is an undefined index — a warning and a null delay, so the final retry fires with no
  backoff at all. Index with `$attempt - 1`.
- WebhookFailure.php:19 — `record()` is called only in the `catch` for `ConnectException`. A
  receiver that answers 500 never reaches it, so the dashboard will show those as successes.
  Record on any non-2xx.

## Consider (1)
- sender.php:31 — `RetryPolicy` duplicates `Support/Backoff::next()`, which already caps and
  jitters. Delete the new class and call the existing one.

## Questions (2)
- worker.php:77 — why does the worker re-read config inside the loop? If it is for hot reload,
  say so in a comment; if not, hoist it.
- worker.php:96 — is `$job->release()` here reachable at all now that line 58 blocks?

## Missing tests
- Retry gives up after the last attempt and records the failure. No test asserts the failure
  path; add one that returns 500 five times and asserts one `WebhookFailure` row.
- Backoff caps at 30 s. No test asserts the cap; assert `next(10) === 30`.

## Not verified
- Callers of `WebhookSender::send()` — no repository access.
- The `integration` CI job was skipped, so nothing here is confirmed against a live receiver.
```

The declined-scope case looks like this:

```
Inputs: diff only. No description, no CI.

## Not a complete review
This diff is 2,140 changed lines across 63 files, over the 800-line threshold. I reviewed three
files and nothing else.

Riskiest files, reviewed: db/migrations/2026_08_14_drop_legacy_tokens.php,
app/Auth/TokenGuard.php, config/session.php.
Not reviewed: the other 60 files, including all of resources/js.

## Blockers (1)
- db/migrations/2026_08_14_drop_legacy_tokens.php:12 — `dropColumn('legacy_token')` with no
  backfill and no down migration. Once this runs, every unmigrated session is unrecoverable.

## Should fix (0)
- None found in the three files reviewed.

## Missing tests
- None assessed. There is no description stating the intended behavior, so "missing" cannot be
  judged.
```

## Edge cases
- **No diff, or an empty diff.** Do not review the description. Reply: "No changes to review." and stop.
- **Description missing.** Run the ritual, and replace step 1 with a single summary line: "No description supplied; intent inferred from the diff and stated below." Then state the inferred intent as a claim the author must confirm. Do not mark description/diff disagreements — you have nothing to compare against.
- **Diff is only lockfiles, generated code, or vendored directories.** Do not comment line by line. Report which lockfiles changed and which direct dependencies moved, and say the generated content was not read.
- **Binary files, minified bundles, or renames with no content change.** List them and skip them. Say "not reviewable as text".
- **No repository access.** Step 3 collapses. Say so in the `Inputs:` line and again under `## Not verified`. Do not name callers, do not estimate blast radius, do not label anything a Blocker on grounds of impact you could not trace. A missing invariant visible inside the diff is still a Blocker.
- **No CI result.** Do not say the change passes. Say "CI not supplied". Treat a green CI you were given as evidence only for the jobs that actually ran; list skipped jobs by name.
- **Truncated or malformed diff** (missing hunk headers, no line numbers, pasted from a UI without context). State that the diff is unusable for line-level comments and ask for `git diff` output. Do not guess line numbers.
- **The PR is a revert.** Review it against the commit it reverts, not against main. The finding to look for is state left behind: migrations already run, feature flags, published events.
- **The PR reformats a file and changes it.** Ask for the split before reviewing. If the split is refused, review only the non-formatting hunks and say which ones you skipped and why.
- **The diff touches a file the input marks as generated** (`*.pb.go`, `schema.sql`, snapshot files). Review the generator input instead, if it is in the diff. If it is not, say the source of the generated change is outside this PR.

## Stop and hand back
**Never approve.** You are the first reader, not the owner. Every run ends with a summary and the blocker list, and no approval verdict of any kind — no "LGTM", no "ready to merge", no checkmark. Approval belongs to the named human reviewer on the PR.

Stop the ritual and hand back to a person on any of these, naming who decides:

- **A credential, private key, token or customer record appears in the diff.** Stop reviewing. Report only that, to the repo's security owner. Say that the secret must be rotated, not just removed from the branch, because it is in the history.
- **The diff changes authentication, authorization, session handling, or crypto.** Publish the review, then hand to the security owner for a second pass before merge. Say explicitly that this skill is not that pass.
- **The diff contains a destructive or irreversible migration** — a dropped column or table, a data backfill with no down path, a deletion job. Route to the person who owns that database. Say what becomes unrecoverable.
- **The change is a hotfix to production during an active incident.** Do not run the full ritual. Report only what would make the incident worse, in under ten lines, and say the review is incomplete by design.
- **The PR is over 800 changed lines** or would produce more than 20 findings. Hand back the scope problem before the code problems: name the three riskiest files, review those, and ask the author to split.
- **The change requires a judgment the input cannot settle** — a product decision, an SLA, a pricing rule, a legal or compliance requirement. State the question and name the role that answers it. Do not resolve it yourself.

## License
MIT
