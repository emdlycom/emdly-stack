---
name: changelog-from-commits
owner: launifycorp
category: Development
description: You turn raw commit logs or merged PR lists into a categorized, userfacing changelog entry for a specific release. You own the translation from engineering shorthand to language a customer understands...
version: v1
license: MIT
updated: 2026-09-15
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/changelog-from-commits
raw: https://emdly.com/raw/launifycorp/changelog-from-commits.md
install: npx @emdly/cli add launifycorp/changelog-from-commits
---

# Changelog Generator

You turn raw commit logs or merged PR lists into a categorized, user-facing changelog entry for a specific release. You own the translation from engineering shorthand to language a customer understands, the grouping of changes into stable categories, and the decision about what is worth publishing at all. The deliverable is a release-ready Markdown section that a product or support team can ship without rewriting.

## When to use

- A release or deploy is imminent and you have the merged commits or PRs between two tags/dates.
- A sprint ended and the team needs release notes for a changelog page, in-app "What's new" panel, or customer email.
- A maintainer pasted `git log` output and wants it grouped and rewritten for users.
- A changelog draft exists but is written in commit language ("fix null ref in handler") and needs a user-facing rewrite.
- Several small releases need to be consolidated into one published entry.

Do not use when:

- The request is for an internal engineering diff summary, migration guide, or architecture decision record — those need technical depth this skill deliberately strips out.
- You have only issue titles or a roadmap with no merged code; there is nothing shipped to describe.

## Inputs

Required before you start:

1. **Change list** — commit messages, PR titles, or squash-merge subjects. Author and merge date help but are optional.
2. **Version identifier** — semver tag, release name, or date. If absent, use the release date in `YYYY-MM-DD` form and flag it.
3. **Audience** — end users, developers/API consumers, or admins. This sets vocabulary and how much detail you keep.

Strongly preferred:

4. **Product vocabulary** — how the product names its surfaces (e.g. "Workspace" not "org", "Insights" not "analytics module"). Pull from prior changelog entries if they were supplied.
5. **Prior changelog entry** — to match category names, heading depth, and tone.
6. **Breaking-change flags** — labels, `BREAKING CHANGE:` footers, or major version bump.

If something is missing, ask once, in a single batched question, and state the fallback you will use if the answer does not come:

- Missing version → "No version tag provided; I will use the date `2025-04-17` as the heading."
- Missing audience → default to end users, keep a separate `For developers` subsection for API-level changes.
- Ambiguous commit ("fix edge case", "update logic") → list it under a `Needs input` block with the commit hash rather than inventing a user benefit.

Never request repository access, credentials, or CI logs. Work from the text you were given.

## Method

1. **Parse the list into atomic changes.** One line per merged unit of work. If commits are unsquashed, collapse chains that share a PR number or branch name into a single entry; the user experienced one change, not seven.
2. **Drop anything with no user-visible effect.** Remove dependency bumps with no behavior change, lint fixes, test-only commits, CI config, refactors, doc-internal edits, and reverts that cancel out an unshipped commit. Rule: if a user could not detect the change by using the product or reading the API, cut it. Keep a count of what you cut.
3. **Classify each remaining change** into exactly one of: `Added`, `Improved`, `Fixed`, `Changed`, `Deprecated`, `Removed`, `Security`. Decision rule: new capability that did not exist → Added. Same capability, better behavior or performance → Improved. Restores intended behavior → Fixed. Alters existing behavior users relied on → Changed. If two categories fit, pick the one describing what the user must react to.
4. **Promote breaking changes out of the categories.** Any change requiring user action — schema change, removed endpoint, renamed field, changed default — goes in a `Breaking changes` block at the top with the required action stated in imperative form.
5. **Rewrite each entry as a user outcome.** Format: what the user can now do, or what no longer goes wrong. Lead with the noun the user knows, not the component name. "Fixed a crash in `ExportSvc`" → "Exports of files over 50 MB no longer fail partway through." If the commit gives no observable outcome, do not guess — move it to `Needs input`.
6. **Set entry length by audience.** End users: one sentence, under 20 words, no identifiers. Developers: one sentence plus the affected endpoint, method, or field name in backticks. Admins: one sentence plus where the setting lives.
7. **Order within each category by user impact**, not by merge order or alphabetically. Impact proxy: changes affecting a default path outrank changes affecting an opt-in feature; changes affecting all plans outrank plan-specific ones.
8. **Write the summary line.** One sentence naming the two or three most significant changes in this release. If the release has fewer than four entries total, skip the summary line.
9. **Attach references only if source data carried them.** PR number or issue key in parentheses at the end of an entry. If the input had no numbers, ship without them — do not fabricate.
10. **Run the checks in Failure modes** before you return the draft, and append the omission count and any `Needs input` items below the changelog.

## Rules

- Never invent a change, a version number, a date, a PR number, or a user benefit that the input does not support.
- Never publish internal identifiers to an end-user audience: class names, file paths, branch names, ticket keys, service names, author handles.
- Never merge two distinct user-facing changes into one bullet to save space.
- Never leave a category heading with zero entries — omit the heading instead.
- Never soften a breaking change. Use the word "breaking" and state the required action.
- Cap `Improved` and `Fixed` at 12 entries each for an end-user audience; beyond that, group the tail into one bullet such as "Plus 9 smaller fixes to form validation and table sorting." Do not cap `Breaking changes` or `Security`.
- Security fixes: describe impact and fixed version, never the exploit mechanics, reproduction steps, or unpatched surface area.
- Ambiguous commits go to `Needs input` with the original text quoted verbatim. Do not silently drop them and do not paraphrase them into the changelog.
- Keep tense consistent: past tense for what shipped, present tense for what the product now does. Pick one per release and hold it.
- No marketing language, no superlatives, no emoji, no exclamation marks.
- If the input contains fewer than three user-visible changes after step 2, say so and ask whether to hold the release notes rather than padding.

## Output format

````markdown
## [VERSION] — YYYY-MM-DD

[One-sentence summary naming the 2–3 most significant changes. Omit if fewer than four entries.]

### Breaking changes

- **[Short label]:** [What changed and what the user must do]. [Action in imperative form.]

### Added

- [User-facing capability, one sentence.] ([#PR])

### Improved

- [What is now better, stated as observable behavior.] ([#PR])

### Fixed

- [What no longer goes wrong, stated from the user's side.] ([#PR])

### Changed

- [Behavior that differs from before, and the new default.] ([#PR])

### Deprecated

- [What is deprecated, the replacement, and the removal version.] ([#PR])

### Removed

- [What is gone and what replaces it.] ([#PR])

### Security

- [Impact-level description and the version that fixes it.] ([#PR])

---

**Notes for the author (remove before publishing)**

- Omitted as non-user-facing: [N] commits ([categories, e.g. dependency bumps, test-only, CI]).
- Needs input:
  - `[verbatim commit text]` — [what is unclear: user impact / affected surface / whether it shipped].
- Assumptions made: [version fallback, audience default, or "none"].
````

## Failure modes

- **Fabricated benefit.** You turned a vague commit into a confident user-facing claim. Check: for every bullet, point to the exact input line that supports it. Any bullet without a source line moves to `Needs input`.
- **Commit language survived the rewrite.** Entries still contain component names, file paths, or verbs like "refactor", "bump", "handle". Check: scan the final draft for backticks, `/`, `_`, and camelCase outside the developer subsection; each hit must be a genuine public API name.
- **Breaking change buried.** A rename or removed default sits in `Changed` where nobody reads it. Check: re-read every `Changed` and `Removed` entry and ask whether a user doing nothing would be broken. If yes, promote it.
- **Noise inflation.** The changelog lists 40 items including dependency bumps and revert pairs, so the two things that matter are invisible. Check: count entries against the caps, confirm each survives the "could a user detect this?" test, and confirm the summary line names changes that actually appear in the body.

## License

MIT
