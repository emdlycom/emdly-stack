---
name: commit-log-to-thread
owner: launifycorp
category: Marketing
description: This skill converts one week of raw commit history into a numbered X/Twitter thread draft that a human can review, edit, and post. You own the translation from implementation detail to uservisible ben...
version: v1
license: MIT
updated: 2026-09-20
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/commit-log-to-thread
raw: https://emdly.com/raw/launifycorp/commit-log-to-thread.md
install: npx @emdly/cli add launifycorp/commit-log-to-thread
---

# Commit Log To Thread

This skill converts one week of raw commit history into a numbered X/Twitter thread draft that a human can review, edit, and post. You own the translation from implementation detail to user-visible benefit: every tweet in the draft must describe something a reader outside the codebase can understand and care about. You do not own posting, scheduling, or approval.

## When to use

- A weekly or sprint-based "what shipped" update is due and the only reliable record is the commit log.
- A product or founder account needs build-in-public content and the week produced concrete changes.
- A release just closed and you need a narrative thread to accompany changelog or release notes.
- A team wants a recurring cadence (e.g. every Friday) of shipping threads generated from git history.
- Someone hands you `git log` output and asks for "a thread about this week."

Do not use when:

- The week's commits are entirely internal refactors, dependency bumps, CI fixes, or reverts with no user-visible effect — say so and recommend skipping the week instead of padding a thread.
- The work is unannounced, embargoed, security-sensitive, or tied to an unreleased customer deal.

## Inputs

Required before you start:

1. **Commit log for the date range.** Preferred format: `git log --since="7 days ago" --pretty=format:"%h|%an|%ad|%s" --date=short --no-merges`. Full commit bodies improve accuracy; subjects alone are workable.
2. **Product name and one-line description** — what the product does and for whom.
3. **Account voice sample** — 2-3 prior posts from the account, or an explicit voice instruction (e.g. "blunt, technical, no hype").
4. **Date range** the thread covers.

Useful if available: linked issue/PR titles, release version number, screenshots or demo links, the primary audience (developers, ops teams, designers), and any items explicitly marked "do not announce."

If any required input is missing, ask for it in one batched request and stop. Do not invent a product description or infer voice from the commit messages themselves. If only the commit log is available and the requester is unreachable, produce the thread with a `[NEEDS INPUT: ...]` marker inline at every point where you would otherwise have guessed.

## Method

1. **Parse and bound the log.** Extract hash, author, date, and subject for each commit. Discard merge commits, revert pairs that cancel out, and commits outside the stated date range. Record the total count of commits processed and discarded — you will report both.

2. **Classify each commit.** Tag every commit as one of: `feature`, `fix`, `performance`, `ux`, `docs`, `infra`, `chore`. Decision rule: if a user of the product would notice the change without reading the repo, it is `feature`, `fix`, `performance`, or `ux`. Everything else is `docs`, `infra`, or `chore`.

3. **Drop the invisible tier.** Remove all `docs`, `infra`, and `chore` commits from thread candidacy. Exception: keep an `infra` commit if it produced a measurable user-facing outcome stated in the commit body (e.g. "cut cold start from 4s to 900ms"), and reclassify it as `performance`.

4. **Cluster into themes.** Group remaining commits by the user-facing outcome they share, not by file or module. Decision rule: if two commits would be described to a customer in the same sentence, they belong to one cluster. Name each cluster with a plain-language outcome phrase ("faster search," "CSV export," "fixed timezone drift on invoices").

5. **Rank clusters.** Score each cluster by: (a) number of users affected, (b) whether it unblocks something previously impossible, (c) size of measurable improvement. Sort descending. If you cannot distinguish (a) across clusters, rank by commit count as a tiebreaker.

6. **Set thread length.** Use 1 hook + N body tweets + 1 closer, where N equals the number of clusters, capped at 6. If you have more than 6 clusters, merge the lowest-ranked ones into a single "also shipped" tweet as the last body tweet. If you have fewer than 2 clusters, return a single standalone post instead of a thread and say why.

7. **Write the hook (tweet 1).** State the concrete headline outcome of the week plus the shipping cadence. Include the date range or week number. No questions, no "🧵 thread incoming" throat-clearing, no promise the body does not deliver. If the top cluster has a number attached, put the number in the hook.

8. **Write body tweets (2..N+1).** One cluster per tweet. Structure each as: what changed → who it helps → the detail that proves it happened. The proof is a metric, a before/after, or a specific behavior — never a commit hash or file path. Match the voice sample's sentence length and formality.

9. **Write the closer.** One line on what is next or how to try it, plus at most one link. If no link or next step was supplied, close with a single factual line about cadence and mark the link slot `[NEEDS INPUT: link]`.

10. **Enforce limits and verify.** Check every tweet is ≤ 280 characters including the `n/N` numbering. Then verify each body tweet against its source commits: if a claim is not supported by a commit subject or body, delete the claim or replace it with `[UNVERIFIED: ...]`. Re-count after edits so `n/N` is correct.

11. **Attach the audit trail.** Below the thread, list each tweet number with the commit hashes it was derived from, plus the excluded-commit count by category. This lets a human check your work in under two minutes.

## Rules

- Never invent metrics, user counts, percentages, or benchmark results. If a number is not in the commit log or supplied inputs, it does not appear in the thread.
- Never expose internal identifiers to readers: no commit hashes, branch names, file paths, ticket IDs, or internal service names in tweet body text. They belong only in the audit trail.
- Never include an author's name without an explicit instruction to credit contributors.
- Never exceed 280 characters per tweet, counting the `n/N` prefix. Shorten the copy; do not drop the numbering.
- Cap the thread at 8 tweets total (hook + 6 body + closer). Longer threads lose readers.
- Never write a tweet whose only content is a chore, dependency bump, lint fix, or test addition.
- Use at most one link in the entire thread, placed in the closer.
- No emoji, no hashtags, no "excited to announce," no "game-changer," no rhetorical questions as openers.
- When commit messages are low-quality (`wip`, `fix stuff`, `.`), do not guess their meaning. Exclude them and report the count as `unreadable` in the audit trail.
- If a commit touches authentication, payments, permissions, or data deletion, flag the corresponding tweet with `[REVIEW: sensitive area]` and leave it for a human to confirm before posting.
- Deliver a draft, never a posted thread. Do not call publishing tools.

## Output format

```
THREAD DRAFT — [Product] — [Start date] to [End date]
Tweets: [N]  |  Commits reviewed: [X]  |  Commits used: [Y]

---

1/[N]
[Hook: headline outcome of the week, with the strongest number if one exists.
Names the product and the date range or week.]

2/[N]
[Cluster 1: what changed → who it helps → the proof.]

3/[N]
[Cluster 2: what changed → who it helps → the proof.]

4/[N]
[Cluster 3: what changed → who it helps → the proof.]

[N]/[N]
[Closer: what is next or how to try it. One link maximum.]

---

AUDIT TRAIL

Tweet 2 — [cluster name] — commits: [hash, hash, hash]
Tweet 3 — [cluster name] — commits: [hash, hash]
Tweet 4 — [cluster name] — commits: [hash]

Excluded:
  infra/chore: [count]
  docs: [count]
  unreadable subjects: [count]
  reverts cancelled: [count]

Flags:
  [REVIEW: sensitive area] — tweet [n], touches [auth/payments/permissions/deletion]
  [NEEDS INPUT: ...] — [what is missing and where]
  [UNVERIFIED: ...] — [claim not supported by a commit]

Character counts: 1/[N]: [n]  2/[N]: [n]  3/[N]: [n]  ...
```

## Failure modes

1. **The thread reads like a changelog.** Symptom: tweets contain verbs like "refactored," "bumped," "migrated," or name modules instead of outcomes. Check: read each body tweet aloud as if speaking to a paying customer who has never seen the repo. If it needs a translation sentence to make sense, rewrite it around the outcome.

2. **Invented specificity.** Symptom: a percentage, latency figure, or user count appears that feels persuasive but has no source. Check: for every number in the draft, point to the exact commit or input that contains it. If you cannot, delete the number or mark it `[UNVERIFIED]`.

3. **Padding a thin week.** Symptom: body tweets about dependency updates, test coverage, or README edits, or two tweets describing the same change in different words. Check: count distinct user-facing clusters before writing. Below 2, downgrade to a single post. Below 1, report "no thread this week" with the excluded-commit breakdown.

4. **Numbering and length drift.** Symptom: `n/N` no longer matches the tweet count after editing, or a tweet silently exceeds 280 characters. Check: as the last action before delivering, recount tweets and recompute every character count including the prefix, and confirm both in the audit trail.

## License

MIT
