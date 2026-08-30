---
name: commit-message-editor
owner: kernelpanic
category: Code review
description: Rewrites a commit message from the diff — imperative subject under 72 characters, a body that explains why, and a footer that links the issue.
version: v3
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/kernelpanic/commit-message-editor
raw: https://emdly.com/raw/kernelpanic/commit-message-editor.md
install: npx @emdly/cli add kernelpanic/commit-message-editor
---

# Commit message editor

Good history is written for the person doing `git blame` at 2 a.m. This skill writes messages for that person: someone who has the diff already, does not have the meeting the change came out of, and needs to know why the line they are staring at exists before they change it.

## When to use
- Before committing, in Claude Code: `git diff --staged` in, message out.
- In a pre-commit hook that rejects messages under a quality bar.
- When rewording during an interactive rebase, one message at a time.
- When a diff has grown past one logical change and you need the split proposed before you commit.

## Input
Required:
- The staged diff (`git diff --staged`). Not the working tree, not the branch diff.
- The last 20 subjects (`git log --oneline -20`), which establish the repo's convention.

Optional, and each changes what the message can say:
- The branch name, for the ticket reference it may carry.
- The issue or ticket text, when the branch name carries a reference. Without it you may cite the reference in a footer but must not describe the issue.
- `CONTRIBUTING.md` or a commit template (`.gitmessage`), which override the inferred house style.

If the last 20 subjects are unavailable, say so and use the repo default: a plain imperative subject with no prefix. Never invent a convention.

## Rules
- **Match the house style first.** Read the 20 subjects. If **12 or more** use Conventional Commits (`feat:`, `fix:`, `chore:`), use it. If 12 or more carry a ticket prefix (`PROJ-12:`), use that. Below 12, the repo has no convention: use a plain imperative subject. Never introduce a convention the repo does not have. [house rule; 12 of 20 is a clear majority, chosen so one contributor's recent streak does not flip the style.]
- **Subject: imperative mood, ≤ 72 characters, no trailing period.** It says what the change does to the system, not what you did: "Add retry to webhook sender", not "Added retries". [72 characters is git's own body wrap width and the point at which common tooling starts truncating a subject in list views; git's `SubmittingPatches` asks for under 50 for the summary, so treat 50 as the target and 72 as the hard cap.]
- **Body: why, not what.** The diff already shows what. Explain the constraint, the bug, the measurement, or the decision a reader could not infer from the code. Wrap at 72 columns. Blank line between subject and body.
- **No body is better than a body that restates the subject.** If you cannot write a why the diff does not already show, ship the subject alone.
- **One logical change per commit.** If describing the diff needs "and", propose a split (see below). Do not write one message covering both.
- **Footer: `Refs PROJ-12` / `Closes #42` only when the reference exists in the input** — in the branch name, the diff, or the supplied ticket. Never derive one from the code.
- **Never mention the tool that wrote the message.** No `Co-authored-by` for an agent, no "generated", no trailer naming a model.
- **Never assert a fact the diff does not carry.** No benchmark numbers, no "fixes the flaky test", no "as discussed" unless the input contains it. Where you need a fact you do not have, write the sentence with a `TODO:` and hand it back rather than inventing.
- Banned in subjects: "various", "misc", "updates", "cleanup", "fixes stuff", "wip", "minor changes". Each names nothing.

## Splitting
Propose a split when any of these is true:
- The diff touches two directories with no shared symbol between them.
- The diff contains both a behavior change and a rename or move of the same code.
- A subject describing the whole diff needs "and" or a comma joining two verbs.
- The diff mixes a dependency bump with code that is not adapting to that bump.

Propose, do not perform. Output one message per proposed commit, each with its file list, in the order they must be applied. Say which files you could not assign and why.

## Output format

A single commit:

```
Add exponential backoff to webhook sender

Webhook targets that return 5xx were retried immediately three times,
which turned a receiver's brief outage into a burst of duplicate calls.
Backoff starts at 1 s and caps at 30 s; the final failure is recorded
so the dashboard can show it.

Refs PROJ-418
```

A split, which is the mandatory shape whenever the "Splitting" rules fire. Return every message in full — a split proposal with an elided message is not usable:

```
## Split proposed: 3 commits
Reason: the diff contains a dependency bump, the code adapting to it, and an
unrelated copy change in the settings page. No shared symbol between
`app/Billing` and `resources/views/settings`.

### 1/3
  files: composer.json, composer.lock

    Upgrade stripe/stripe-php 13.18 to 14.2

    14.0 moved `Charge::create` to `PaymentIntent`; the adapting code is in
    the next commit, so this one alone leaves the build red. Apply both or
    neither.

    Refs PROJ-902

### 2/3
  files: app/Billing/StripeGateway.php, app/Billing/RefundJob.php,
         tests/Billing/StripeGatewayTest.php

    Move charge creation to PaymentIntent

    stripe/stripe-php 14 removed `Charge::create`. PaymentIntent needs an
    explicit confirmation step, so `RefundJob` now looks up the intent by id
    instead of the charge id; existing refunds keep working because the
    charge id is still stored on the row.

    Refs PROJ-902

### 3/3
  files: resources/views/settings/billing.blade.php

    Rename "Card on file" to "Payment method"

    (no body: the change is a label rename and the reason is not recorded in
    the input)

## Unassigned
- .idea/workspace.xml — editor state, does not belong in any commit. Unstage it.
```

The empty case:

```
## No message
The staged diff is empty. Nothing to describe. Run `git add` first.
```

The withheld case, when a required fact is missing:

```
Cap session lifetime at 12 hours

TODO: state why 12 hours. The value appears only as a literal in
config/session.php and no ticket, comment or test in the input explains it.
Supply the reason or commit with the subject alone.
```

## Edge cases
- **Merge and revert commits** keep git's default subject. Add a body only when the reason is not obvious from that subject — for a revert, name what broke and where it was observed.
- **A diff of only generated files** (lockfiles, `*.pb.go`, snapshots, minified bundles) gets a one-line subject naming the generator and the trigger, and no body. Do not read the generated content.
- **Empty diff.** Emit the "No message" block above and stop. Do not fall back to the unstaged diff, the branch diff, or the last commit — each describes a different change than the one about to be recorded.
- **Fewer than 20 commits in the repo.** Use every subject there is and say how many you read: "style inferred from 6 subjects". Below **5 subjects** there is no convention to infer: use a plain imperative subject, and say that the style was defaulted, not inferred. [house rule.]
- **The 20 subjects are split**, with no style at 12 or more — for example 9 Conventional and 8 ticket-prefixed. Use the style of the most recent 5 subjects and say which you followed and that the repo is inconsistent.
- **Whitespace- or formatter-only diff.** One-line subject, name the tool: "Reformat app/ with prettier 3.3". Never let a behavior change hide in one of these; if the diff contains both, that is a mandatory split.
- **The diff is too large to read in full** (over ~2,000 changed lines, or the input is truncated). Do not summarize what you did not read. Report which files you read, and either write the message from those and label it partial, or ask for a smaller staged set. Say which. [house rule; the threshold exists so the message never claims coverage it does not have.]
- **The branch name carries a reference the ticket text does not confirm** (`fix/PROJ-12-typo` with no ticket supplied). Put `Refs PROJ-12` in the footer. Do not describe the ticket.
- **A revert of a revert**, or a cherry-pick. Keep the original subject and add one body line naming where it came from and why it is back.
- **The diff only deletes code.** Say what is gone and what now handles the case, or state that nothing does. A deletion with no successor named is the message a reader will curse in two years.

## Stop and hand back
The message becomes permanent, public history the moment it is committed, and in a pre-commit hook it ships with no further review. Stop and hand to a person on any of these:

- **The diff contains a credential, key, token, or customer record.** Do not write a message. Report the file and line to the repo's security owner and say the commit must not be made — a secret is not fixed by a later commit, only by rotation and a history rewrite.
- **The diff changes authentication, authorization, crypto, or a permission check** and the input does not say why. Do not infer the security intent. Hand back with the question; a wrong "why" in the history is worse than none.
- **The diff changes a LICENSE, NOTICE, or third-party attribution file.** Route to whoever owns licensing. Do not describe the legal effect.
- **The diff deletes or rewrites customer data**, or contains a migration that is not reversible. Name what is unrecoverable and hand to the database owner before the commit is made.
- **The message would need a fact you cannot verify** — a benchmark, an incident id, an agreement, a customer name. Emit the `TODO:` form above and hand back. Never fill it with a plausible guess.

## License
MIT
