---
name: stale-branch-cleaner
owner: launifycorp
category: Development
description: Audits the branches of a git repository, ranks them by how likely they are to be dead, presents the candidates for deletion, and deletes only what the user explicitly confirms — locally, on the remote...
version: v2
license: MIT
updated: 2026-09-03
recommended: true
security_checked: true
url: https://emdly.com/skills/launifycorp/stale-branch-cleaner
raw: https://emdly.com/raw/launifycorp/stale-branch-cleaner.md
install: npx @emdly/cli add launifycorp/stale-branch-cleaner
---

# Stale Branch Cleaner

Audits the branches of a git repository, ranks them by how likely they are to be dead, presents the candidates for deletion, and deletes only what the user explicitly confirms — locally, on the remote, or both.

Deletion is destructive and, on a remote, effectively public. This skill NEVER deletes anything before an explicit confirmation, and never widens the scope it was given.

## When to use

- "clean up my branches", "which branches can I delete", "prune stale branches"
- "my repo has 60 branches and I don't know which are dead"
- Before a repo handover, an audit, or a release cleanup.

Do not use it to delete a branch the user already named — that is a one-line git command, not an audit.

## Inputs to establish first

Ask only for what you cannot detect. Detect the rest.

- **Repository path** — default: the current working directory. Refuse to continue if it is not a git work tree.
- **Staleness threshold** — default: **90 days** since the last commit. State the default; let the user override.
- **Remotes** — `git remote -v`. A fork checkout has `origin` and `upstream`, and they are not interchangeable: deleting on the wrong one either fails or destroys the wrong branch. If there is more than one, ask which is meant and never assume `origin`.
- **Protected branches** — never proposed for deletion under any circumstance:
  `main`, `master`, `develop`, `dev`, `staging`, `production`, `prod`, `release`, `release/*`, `HEAD`, the current branch, and the remote's default branch (`git symbolic-ref refs/remotes/origin/HEAD`). Add anything the user names.

## Steps

1. **Refresh the view of the remote** (read-only, safe):
   `git fetch --all --prune --quiet`
   `--prune` only cleans up local *tracking refs* for branches already deleted on the remote; it deletes no branches. Say so if the user asks.

2. **Collect the facts** in one pass, so every judgement rests on real data:

   ```
   git for-each-ref --sort=committerdate \
     --format='%(refname:short)|%(committerdate:iso8601)|%(authorname)|%(upstream:track)' \
     refs/heads refs/remotes
   ```

   For each branch also determine:
   - days since last commit — from the date above;
   - **merged or not**: `git branch --merged <default-branch>` (local) and `git branch -r --merged origin/<default-branch>` (remote);
   - **unmerged commits that would be lost**: `git log --oneline <default-branch>..<branch> | wc -l`;
   - whether a local branch still has a live remote counterpart, and vice versa.

3. **Catch the squash-merged branches.** This is the trap that makes a naive audit wrong on most modern repos: `git branch --merged` follows ancestry, and a **squash merge** or a **rebase merge** creates new commits with new SHAs. The branch's own commits are never ancestors of the default branch, so its work is fully shipped and it still reports as unmerged with N commits ahead.

   Treating those as `STALE` buries the safest deletions in the scariest pile. Before classifying, test each apparently-unmerged branch:

   ```
   git cherry -v <default-branch> <branch>
   ```

   Lines starting `-` are commits whose change is already present upstream (matched by patch-id); `+` means genuinely absent. A branch whose lines are all `-` is **squash-merged** — equivalent in content, unreachable by ancestry.

   Where a forge is reachable, its answer beats inference: `gh pr list --state merged --head <branch>` (or `glab mr list`) proves the branch was merged and names the PR. Use it when available and say which source the verdict came from.

4. **Classify** each non-protected branch:

   | Class | Meaning | Default proposal |
   |---|---|---|
   | `MERGED` | fully merged into the default branch by ancestry | propose deletion at any age |
   | `SQUASHED` | content present upstream (`git cherry` all `-`, or a merged PR) | propose deletion, state the evidence |
   | `STALE` | older than the threshold, has genuinely absent commits | propose, flagged with the commit count |
   | `GONE` | local branch whose upstream is gone (`[gone]`) | propose local deletion |
   | `ACTIVE` | newer than the threshold | keep, do not list among candidates |

   Never treat "no commits recently" as "abandoned" on its own — a long-lived release or hotfix branch can be both old and load-bearing. Anything with unmerged work is flagged, not quietly bundled in.

5. **Check what a deletion would disturb** before proposing anything:
   - **Worktrees** — `git worktree list`. A branch checked out in another worktree cannot be deleted and, more importantly, someone is working in it.
   - **Open PRs** — `gh pr list --head <branch>`. An open PR means the branch is alive whatever its age; exclude it from the default proposal.
   - **Tags** — a tag pointing at a branch's tip keeps the commits reachable, so deleting that branch loses nothing. Worth saying, because it turns a scary deletion into a safe one.
   - **Remote protection rules** — a protected branch on the forge will refuse the push. Predict it rather than discovering it in an error.

6. **Present the list and stop.** Group by class, most confident deletions first. For each branch show: name, age, last author, merged verdict and how it was reached, unmerged commit count, and where it exists (local / remote / both). Show the totals. Then state explicitly that nothing has been deleted yet.

7. **Ask for the scope** — this is a separate question from *which* branches, and it has no default:
   - **local only** — safe, reversible, affects nobody else;
   - **remote only** — affects everyone who uses the repo;
   - **both**.

   Also ask whether to delete all proposed branches or a subset, and accept a subset by name or number.

8. **Confirm before acting.** Restate the exact branch names, the exact remote, and the exact scope, then wait for a clear yes. Treat silence, "ok whatever", or an ambiguous reply as "no". If the scope includes a remote, say plainly that remote deletion affects other people.

9. **Delete only what was confirmed**, one branch at a time, reporting each result:
   - local, merged: `git branch -d <branch>`
   - local, squashed or unmerged (only after the user confirmed seeing the evidence): `git branch -D <branch>`
   - remote: `git push <remote> --delete <branch>`

   Print every tip SHA **before** deleting, in one recoverable block. Recovery is `git checkout -b <name> <sha>`, and it works while the commit is still reachable: locally the reflog holds it for a limited window (`gc.reflogExpireUnreachable`, 30 days by default — quote the repo's actual setting, not an assumption), and on the remote only if someone still holds a copy. A deleted remote branch whose commits exist nowhere else is gone for good.

10. **Report**: deleted, skipped, failed (with the error). If any deletion failed — branch protection, no push rights, an open PR, a checked-out worktree — say which and why, and do not retry with a more forceful command.

## Rules

- Never delete without an explicit confirmation naming the scope. No exceptions, including when the user says "just clean it up".
- Never `git push --delete` a protected branch, even if asked; say why and stop.
- Never force-delete (`-D`) a branch with genuinely unmerged commits unless the user confirmed *after* seeing the commit count.
- Never call a branch unmerged on ancestry alone — run the patch-id check first, or the report is wrong on every squash-merging repo.
- Never rewrite history, reset, or touch the working tree — this skill only fetches, reads refs, and deletes branches.
- Never delete a branch checked out in a worktree, or one with an open PR, as part of a bulk action.
- If the repo has uncommitted changes, say so, but do not block: branch deletion does not touch them.
- Never run `git gc`, `git prune`, or `git reflog expire`. They are what turns a recoverable deletion into a permanent one.

## Output format

```
Repository: <path>   Default branch: <name>   Remote: origin   Threshold: <N> days
Scanned: <N> local, <N> remote   Protected: <list>   Worktrees: <n>

MERGED — safe to delete (<n>)
  feature/login-form      142d   J. Novak    ancestry        local+origin

SQUASHED — shipped, invisible to --merged (<n>)
  feature/cart-rewrite     96d   M. Dvorak   PR #412 merged  local+origin
  fix/header-overflow     120d   J. Novak    patch-id match  local only

STALE — genuinely unmerged work (<n>)
  spike/graphql           210d   M. Dvorak   3 commits ahead origin only

GONE — upstream deleted (<n>)
  fix/typo                 88d   J. Novak    [gone]          local only

EXCLUDED
  feature/checkout-v2   open PR #488
  hotfix/tls            checked out in worktree ../hotfix

KEEPING: <n> active branches

Nothing deleted yet. Delete which branches, and where — local only, origin only, or both?
```

After deletion:

```
Recovery block (save this):
  feature/cart-rewrite  a3f91c2      fix/header-overflow  7d20e11
  git checkout -b <branch> <sha>   — local reflog holds unreachable commits for <N> days

Deleted (local): <list>
Deleted (origin): <list>
Skipped: <list with reasons>
Failed: <list with errors>
```

## License

MIT
