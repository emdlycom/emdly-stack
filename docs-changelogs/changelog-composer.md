---
name: changelog-composer
owner: shiplog
category: Docs & changelogs
description: Turns a week of merged PRs into a changelog humans read — grouped by impact, written for users, breaking changes on top.
version: v3
license: MIT
updated: 2026-08-25
recommended: false
security_checked: true
url: https://emdly.com/skills/shiplog/changelog-composer
raw: https://emdly.com/raw/shiplog/changelog-composer.md
install: npx @emdly/cli add shiplog/changelog-composer
---

# Changelog composer

A changelog is documentation for people who did not read the PRs. This skill writes it for them.

## When to use
- Weekly or per release, on the list of merged PRs (title, body, labels, author, link).
- For a product changelog page, release notes e-mail, or the `CHANGELOG.md`.

## Process
1. **Drop what users cannot see:** refactors, CI, test-only, dependency bumps without behavior change. Count them in one line at the end ("+14 internal changes").
2. **Group by impact:** Breaking → New → Improved → Fixed. Not by component, not by author.
3. **Rewrite each entry for the user's job.** Start with what they can now do or what no longer happens; the PR title is input, not output. "Feed import: rows with a missing GTIN are now flagged instead of skipped" — not "Handle null GTIN in importer".
4. **Breaking changes** carry what breaks, who is affected, and the migration step. These go first, always.
5. Link every entry to its PR or docs page.

## Rules
- User's vocabulary, present tense, one sentence per entry (two for breaking).
- No author names in the product changelog; keep them in the internal one if asked.
- Never claim an improvement the PR does not show ("faster" needs a number in the PR).
- Keep the whole thing scannable: under 20 entries; if more, split by area with headings.
- Date and version at the top; the date is the release date, not today, if they differ.

## Output format
```
## 2026-08-25 · v1.14

### Breaking
- **API tokens are now per agent.** Account-level tokens stop working on 1 Sep. Create a token on your agent's page and replace it in your integrations. ([#212](…))

### New
- Download any skill version as a `.md` from the Versions tab. ([#198](…))

### Improved
- Search ignores punctuation, so "pr-review" and "pr review" find the same skills. ([#204](…))

### Fixed
- The install counter no longer counts a cancelled download. ([#209](…))

+14 internal changes.
```

## License
MIT
