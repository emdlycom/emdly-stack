---
name: changelog-composer
owner: shiplog
category: Docs & changelogs
description: Turns a week of merged PRs into a changelog humans read — grouped by impact, written for users, breaking changes on top.
version: v4
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/shiplog/changelog-composer
raw: https://emdly.com/raw/shiplog/changelog-composer.md
install: npx @emdly/cli add shiplog/changelog-composer
---

# Changelog composer

A changelog is documentation for people who did not read the PRs. The reader is a customer who opened the page to find out whether anything they depend on changed, so every entry has to say what they can now do or what no longer happens, and nothing may appear that the merged PRs do not prove. Breaking changes and the dates attached to them are the part of this document that can hurt someone, so they leave this skill as a draft with an open question, never as published copy.

## When to use
- Weekly, or per release, on the merged PRs since the last published entry.
- To draft a product changelog page, a release-notes email, or a `CHANGELOG.md` section.
- To produce the internal changelog (authors kept) alongside the customer one.
- When a release is already out and someone asks what to tell customers.
- Not for a launch or marketing post. That needs claims this skill refuses to make.

## Input

Required:
- **PR list** — for every merged PR: title, body, labels, author, merge date, link. A `gh pr list --state merged --json number,title,body,labels,author,mergedAt,url` dump is the expected shape.
- **Range** — the tag range or date window the list covers, plus the previously published version and date, so the reader can tell what this entry replaces.
- **Version and release date** — the date users get the release. If that is not today, use it anyway.
- **Audience** — one of `product page`, `email`, `CHANGELOG.md`, `internal`.

Optional, and each one changes what you may write:
- **Docs base URL** — without it, entries link to PRs only.
- **The previous published entry** — for heading style, casing and voice.
- **A named owner** for breaking changes, so `## Stop and hand back` has somewhere to go.

PR titles alone are not enough input. Step 3 rewrites the entry from what the PR body and diff show; with titles only, say `bodies not supplied — entries are titles restated` at the top and do not claim behaviour the title does not state.

## The pass, in order

1. **Split user-visible from internal.** Internal: refactors, CI, tests, build, formatting, dependency bumps with no behaviour change, and reverts of a change that never shipped. Everything else is user-visible until you can point to the line in the PR body that says otherwise. Count the internal ones; they become the last line.
2. **Bucket by impact:** Breaking → New → Improved → Fixed. Never by component, author, or PR number. An entry that is both new and breaking goes in Breaking once, not in both.
3. **Rewrite each entry from the reader's job.** Open with what they can now do, or what no longer happens. The PR title is input, not output. Write "Feed import flags rows with a missing GTIN instead of skipping them", not "Handle null GTIN in importer".
4. **Give every breaking entry four parts:** what breaks, who is affected, the migration step, and the date it takes effect. If any of the four is missing from the PR, the entry is not finishable — hold it (see `## Stop and hand back`).
5. **Link every entry** to its PR, and to the docs page when a docs URL exists.
6. **Count and reconcile.** Entries listed + internal count must equal the number of merged PRs in the range. Print both numbers. If they do not tie, the split is wrong; fix it before writing anything else.

## Rules

- The reader's vocabulary, present tense, one sentence per entry. Breaking entries get up to three: what breaks, the migration, the date.
- Never claim a measurement the PR does not contain. "Faster" needs a number from the PR; otherwise write what changed, not that it improved.
- No author names on a customer changelog. Keep them on the internal one when asked.
- Under 20 entries in one block (house rule, for scanning). Past 20, split by product area with `###` headings and keep the impact order inside each.
- Version and date at the top. The date is the release date.
- A held entry stays visible in the draft as a `HOLD` block. Never drop it and never soften it into a non-breaking entry.

## Output format

Version and date, then the impact buckets in order, then the internal count, then the holds.

```
## 2026-08-25 · v1.14
Range: v1.13 (2026-08-18) → v1.14. 31 PRs merged: 5 user-visible, 26 internal.

### Breaking
- **API tokens are now per agent.** Account-level tokens keep working until the cutover
  date below. Create a token on the agent's page and replace it in your integrations.
  Effective date: HELD — see below. ([#212](https://github.com/shiplog/app/pull/212))

### New
- Download any version of a skill as a `.md` file from the Versions tab.
  ([#198](https://github.com/shiplog/app/pull/198) ·
  [docs](https://docs.shiplog.dev/versions))

### Improved
- Search ignores punctuation, so "pr-review" and "pr review" return the same skills.
  ([#204](https://github.com/shiplog/app/pull/204))

### Fixed
- The install counter no longer counts a download the user cancelled.
  ([#209](https://github.com/shiplog/app/pull/209))
- Skill pages no longer show a blank category when the category was renamed.
  ([#217](https://github.com/shiplog/app/pull/217))

+26 internal changes.

--- HOLD: not for publication until confirmed ---
HOLD 1 — breaking entry #212, effective date.
  PR body says "account tokens stop working on 1 Sep". Not published until confirmed.
  Confirm with: Priya Raman (owner, platform-api).
  Confirm: (a) 1 Sep 2026 is the real cutover, (b) affected customers have been mailed
  separately, (c) the migration doc at /docs/tokens is live.
  Publishing this line tells every customer their integrations break in 7 days.
```

Counts tie: 1 + 1 + 1 + 2 = 5 entries, 5 + 26 = 31 PRs.

A week with nothing user-visible is a complete result, not a failure:

```
## 2026-09-01 · no release
Range: v1.14 (2026-08-25) → HEAD. 11 PRs merged: 0 user-visible, 11 internal.

No user-visible changes this week.

+11 internal changes (9 dependency bumps, 1 CI, 1 test-only).

Do not publish an empty entry to the product page. Two options, both deliberate:
skip the week, or publish the "no user-visible changes" line above so subscribers know
the page is current. Ask the changelog owner which; do not decide it here.
```

## Edge cases

- **No PR list.** The skill has no other source. Report `no PR list supplied` and stop. Do not read the commit log instead: commit messages do not carry labels or bodies and the split in step 1 becomes guesswork.
- **PR list with titles only.** Runs, with `bodies not supplied — entries are titles restated` at the top. Every breaking candidate is held, because step 4's four parts cannot come from a title.
- **Zero user-visible changes.** Use the empty output above. Never promote an internal change to fill the page.
- **Zero PRs in the range.** Report `no PRs merged in this range` and check the range with whoever supplied it before writing anything.
- **More than 200 PRs**, or a list too large to read in full. Do not sample. Process in merge-date order in blocks of 50, keep a running internal count, and reconcile once at the end against the total. Say in the output that it was processed in blocks.
- **A PR whose body contradicts its title.** The body wins, and the entry carries `(title and body disagree — body used)` in the draft for the reviewer.
- **A PR labelled `breaking` with no migration step.** Hold it. An unmigrated breaking change is not publishable copy.
- **A revert of something already published.** It is a user-visible entry under Fixed, worded as the current state ("X works again"), and it links both PRs.
- **No docs URL supplied.** Entries link to PRs only. Do not guess a docs path.
- **No previous entry supplied.** Write in the house voice above and say `no previous entry supplied — heading style not matched`.
- **Dates that disagree** (merge date, tag date, release date). The release date is the entry date. If no release date was supplied, hold the whole entry rather than dating it today.

## Stop and hand back

Stop, publish nothing, and name who decides. This document reaches customers without another edit, so these are hard stops, not cautions.

- **Any breaking-change entry.** Every breaking entry is held for a named human, always, even when the PR looks complete. Hand back: the drafted entry, the PR link, and the three confirmations in the example above. Whoever owns the affected surface confirms.
- **A date at which something stops working** — deprecation, cutover, sunset, end of support. Never publish a date sourced only from a PR body. Hand back to the owner named in Input with the exact sentence you would publish.
- **A security-relevant fix.** Do not describe the vulnerability, the affected versions, or the exploit path. Hold the entry and hand it to whoever runs security disclosure; they decide the wording and the timing.
- **Anything about pricing, plan limits, quotas, or billing.** Hand to the person who owns pricing. A PR that changes a limit is not authority to announce it.
- **A change to data retention or deletion**, or to what is collected. Hand to whoever signs off on the privacy policy.
- **No named owner supplied** and the release contains any of the above. Emit the draft with the holds intact and stop. Do not publish the non-breaking entries either: a changelog that quietly omits the breaking change is worse than no changelog.

## License
MIT
