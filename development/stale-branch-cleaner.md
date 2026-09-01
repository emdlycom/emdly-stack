---
name: stale-branch-cleaner
owner: launifycorp
category: Development
description: Audits the branches of a git repository, ranks them by how likely they are to be dead, presents the candidates for deletion, and deletes only what the user explicitly confirms — locally, on the remote...
version: v1
license: MIT
updated: 2026-09-01
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
- **Protected branches** — never proposed for deletion under any circumstance:
  `main`, `master`, `develop`, `dev`, `staging`, `production`, `prod`, `release`, `release/*`, `HEAD`, the current branch, and the remote's default branch. Add anything the user names.

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

3. **Classify** each non-protected branch:

   | Class | Meaning | Default proposal |
   |---|---|---|
   | `MERGED` | fully merged into the default branch | propose deletion at any age |
   | `STALE` | older than the threshold, has unmerged commits | propose, flagged with the commit count |
   | `GONE` | local branch whose upstream is gone (`[gone]`) | propose local deletion |
   | `ACTIVE` | newer than the threshold | keep, do not list among candidates |

   Never treat "no commits recently" as "abandoned" on its own — a long-lived release or hotfix branch can be both old and load-bearing. Anything with unmerged work is flagged, not quietly bundled in.

4. **Present the list and stop.** Group by class, most confident deletions first. For each branch show: name, age, last author, merged status, unmerged commit count, and where it exists (local / origin / both). Show the totals. Then state explicitly that nothing has been deleted yet.

5. **Ask for the scope** — this is a separate question from *which* branches, and it has no default:
   - **local only** — safe, reversible, affects nobody else;
   - **origin only** — affects everyone who uses the repo;
   - **both**.

   Also ask whether to delete all proposed branches or a subset, and accept a subset by name or number.

6. **Confirm before acting.** Restate the exact branch names and the exact scope, then wait for a clear yes. Treat silence, "ok whatever", or an ambiguous reply as "no". If the scope includes `origin`, say plainly that remote deletion affects other people and that recovery requires someone to still hold the commit.

7. **Delete only what was confirmed**, one branch at a time, reporting each result:
   - local, merged: `git branch -d <branch>`
   - local, unmerged (only if the user confirmed knowing the commit count): `git branch -D <branch>`
   - remote: `git push origin --delete <branch>`

   Before deleting anything unmerged, print its tip SHA so it can be recovered with `git checkout -b <name> <sha>`. Recovery is possible while the commit is still in the reflog or held by another clone.

8. **Report**: deleted, skipped, failed (with the error). If any deletion failed — branch protection rules, no push rights, an open PR — say which and why, and do not retry with a more forceful command.

## Rules

- Never delete without an explicit confirmation naming the scope. No exceptions, including when the user says "just clean it up".
- Never `git push --delete` a protected branch, even if asked; say why and stop.
- Never force-delete (`-D`) a branch with unmerged commits unless the user confirmed *after* seeing the commit count.
- Never rewrite history, reset, or touch the working tree — this skill only fetches, reads refs, and deletes branches.
- If the repo has uncommitted changes, say so, but do not block: branch deletion does not touch them.
- If a branch has an open pull request, mention it and exclude it from the default proposal.

## Output format

```
Repository: <path>   Default branch: <name>   Threshold: <N> days
Scanned: <N> local, <N> remote   Protected: <list>

MERGED — safe to delete (<n>)
  feature/login-form      142d   J. Novak    merged      local+origin
  ...

STALE — unmerged work (<n>)
  spike/graphql           210d   M. Dvorak   3 commits ahead   origin only
  ...

GONE — upstream deleted (<n>)
  fix/typo                 88d   J. Novak    [gone]      local only

KEEPING: <n> active branches

Nothing deleted yet. Delete which branches, and where — local only, origin only, or both?
```

After deletion:

```
Deleted (local): <list>
Deleted (origin): <list>
Skipped: <list with reasons>
Failed: <list with errors>
Recovery: <branch> tip was <sha> — git checkout -b <branch> <sha>
```

## License

MIT
