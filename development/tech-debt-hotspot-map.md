---
name: tech-debt-hotspot-map
owner: launifycorp
category: Development
description: You rank files and modules in a codebase by the product of change frequency (churn) and structural complexity, producing a ranked hotspot table plus a short remediation brief. The outcome you own is a...
version: v1
license: MIT
updated: 2026-09-20
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/tech-debt-hotspot-map
raw: https://emdly.com/raw/launifycorp/tech-debt-hotspot-map.md
install: npx @emdly/cli add launifycorp/tech-debt-hotspot-map
---

# Tech Debt Hotspot Map

You rank files and modules in a codebase by the product of change frequency (churn) and structural complexity, producing a ranked hotspot table plus a short remediation brief. The outcome you own is a defensible, reproducible list of the 10-25 places where refactoring effort will pay back fastest, with the evidence (commit counts, author spread, complexity metrics) attached to each entry so a reviewer can verify or overturn your ranking without rerunning the analysis.

## When to use

- A team is planning a refactoring quarter or tech-debt budget and needs to pick targets instead of arguing from memory.
- Defect rates or incident post-mortems keep pointing at the same area and someone asks "is this actually our worst code?"
- A new maintainer or consultant is onboarding to an unfamiliar repo and needs a map of where the risk lives.
- Before a large migration (framework upgrade, language version, module extraction) to find which files will fight back hardest.
- A release slowed measurably and leadership wants an evidence-backed answer about where friction accumulated.

Do not use when:

- The repository has fewer than ~150 commits or under ~3 months of history — churn signal is noise at that volume; do a manual read instead.
- The request is "review this pull request" or "find bugs in this file." This skill measures accumulation over time, not correctness of current code.

## Inputs

Before starting you need:

1. **Repository access** with full git history (not a shallow clone). Verify with `git rev-parse --is-shallow-repository`; if it returns `true`, ask for a full clone or run `git fetch --unshallow`.
2. **Analysis window** — a start date or commit range. Default to the last 12 months if unspecified.
3. **Language/stack** of the codebase, so you pick a complexity tool that actually parses it.
4. **Exclusion list** — vendored code, generated files, lockfiles, migrations, test fixtures, build output.
5. **Known non-signals** — any bulk reformat, license-header sweep, or mass rename commits that would inflate churn.

If missing, ask for them in this order and proceed with stated defaults rather than blocking:

- Missing exclusion list → derive one from `.gitignore`, `vendor/`, `node_modules/`, `dist/`, `build/`, `*.generated.*`, `*_pb.go`, `*.min.js`, and state it in the output.
- Missing bulk-commit list → detect candidates yourself (commits touching >100 files or changing >5,000 lines) and list them for confirmation.
- Missing stack info → infer from file extensions and `git ls-files` distribution.
- Missing analysis window → use 12 months and say so.
- No complexity tool available for the language → fall back to indentation-based complexity and label the column clearly as a proxy.

## Method

1. **Confirm history depth and window.** Run `git log --oneline --since="<start>" | wc -l`. If the result is under 150 commits, stop and report that the sample is too small rather than producing a ranking with false precision.

2. **Build the exclusion filter first.** Apply the exclusion list and drop any path matching generated/vendored patterns. If excluded files would have entered the top 25, note them in an "Excluded but high-churn" appendix — sometimes a generated file signals a real problem upstream.

3. **Compute churn per file.** Run `git log --since="<start>" --numstat --no-merges --pretty=format:"%H|%an|%ad"` and aggregate per path: commit count, lines added, lines deleted, distinct author count, date of last change. Follow renames with `--follow` on candidate files only (it is expensive); if a file has fewer than 5 commits, skip rename tracing.

4. **Neutralize bulk commits.** Exclude any commit touching more than 100 files or more than 5,000 net lines unless the requester confirmed it was real work. Recompute churn after exclusion and record how many commits were dropped.

5. **Measure complexity per file.** Use the best available tool for the stack: `radon cc -s` (Python), `eslint` complexity rule or `escomplex` (JS/TS), `gocyclo` (Go), `lizard` (C/C++/Java/multi-language), `rubocop -f json` ABC size (Ruby). Record the maximum function complexity and total file complexity. If no tool parses the language, compute the proxy: mean leading-whitespace depth × logical line count, and mark the column `(proxy)`.

6. **Normalize both axes.** Convert churn commit-count and complexity to percentile ranks across the analyzed file set (0-100). Use percentiles, not raw values, so one 4,000-line file does not dominate the map.

7. **Score and rank.** `hotspot_score = churn_percentile × complexity_percentile / 100`, rounded to integer. Rank descending. Break ties by author count (more authors = higher risk), then by recency of last change.

8. **Classify each top entry.** Assign exactly one label using this decision rule:
   - `HOTSPOT` — churn ≥ 75th pct AND complexity ≥ 75th pct. Primary refactor targets.
   - `BRITTLE` — complexity ≥ 75th pct, churn < 50th pct. Dangerous but stable; touch only when required.
   - `THRASH` — churn ≥ 75th pct, complexity < 50th pct. Likely a config, constants, or coordination point, not a code-quality problem. Investigate why it changes, do not refactor.
   - `WATCH` — anything else in the top 25.

9. **Roll up to modules.** Aggregate scores by directory at the depth where the repo's own structure has meaning (usually 2 levels below root, or the package/module boundary the language defines). Report the top 5 modules by summed hotspot score and by mean score — they answer different questions and often disagree.

10. **Sample-verify the top 3.** Open each of the top three files and write one concrete sentence of evidence: the longest function and its length, the deepest nesting level, the number of distinct responsibilities visible. If a file's code does not look bad, say so explicitly and demote it — metrics that contradict reading are metrics that are wrong.

11. **Write the remediation brief.** For each `HOTSPOT`, name one specific first move (extract function X, split by responsibility Y, add characterization tests around Z) and a rough size in engineer-days. If you cannot name a specific move, write "needs investigation" rather than a generic suggestion.

## Rules

- Never present a ranking without stating the analysis window, the commit count analyzed, the number of files scored, and the complexity tool used. An unlabeled ranking is unusable a month later.
- Never claim a causal link between hotspot score and defects unless you were given defect or incident data to join against. Say "correlates with maintenance cost" not "causes bugs."
- Never include test files in the main ranking. Test churn is expected. Report them in a separate section if their scores are extreme.
- Never recommend rewriting a `THRASH` file. High churn with low complexity is almost always a coordination or configuration issue.
- Do not fabricate complexity numbers when a parser fails. Mark the file `complexity: unavailable`, exclude it from percentile computation, and list it separately.
- Cap the main table at 25 rows. Longer lists do not get acted on.
- Respect a repository-size limit: if the analysis would score more than 20,000 files, restrict to the top 2,000 by churn before computing complexity, and disclose the cutoff.
- If more than 20% of files fail complexity parsing, stop and report a tooling problem instead of publishing a partial map.
- Do not modify the repository. Read-only operations only; no commits, no branches, no file writes inside the repo.

## Output format

````markdown
# Tech Debt Hotspot Map — <repo name>

**Window:** <start date> to <end date> · **Commits analyzed:** <n> (<m> bulk commits excluded)
**Files scored:** <n> · **Complexity tool:** <tool + version> · **Generated:** <date>
**Exclusions:** <patterns applied>

## Top hotspots

| # | File | Label | Score | Churn (commits) | Authors | Complexity (max/total) | Last changed |
|---|------|-------|-------|-----------------|---------|------------------------|--------------|
| 1 | path/to/file.ext | HOTSPOT | 92 | 61 | 9 | 24 / 310 | 2026-01-14 |
| 2 | ... | ... | ... | ... | ... | ... | ... |

## Module rollup

| Module | Files | Summed score | Mean score | Dominant label |
|--------|-------|--------------|------------|----------------|
| src/billing | 14 | 480 | 34 | HOTSPOT |

## Verified top 3

- **<file>** — <one sentence of concrete evidence from reading the code>
- **<file>** — <evidence>
- **<file>** — <evidence; include demotions here if metrics contradicted the read>

## Remediation brief

| File | First move | Est. size | Prerequisite |
|------|-----------|-----------|--------------|
| path/to/file.ext | Extract <function> into <module>; add characterization tests first | 3-5 d | Test harness for <component> |

## Do not refactor

- **<file>** (THRASH) — <why the churn is structural, not quality-driven>

## Data caveats

- <files with unavailable complexity>
- <excluded-but-high-churn generated files>
- <any cutoff, proxy metric, or confirmation still pending>
````

## Failure modes

1. **Bulk commits dominate the ranking.** A formatter run or dependency bump makes 300 files look hot. *Check:* before publishing, list the top 10 files' largest single commit; if any file's churn is >50% from one commit, that commit is suspect — verify it and re-run step 4.

2. **Generated or vendored code ranks first.** Protobuf output, bundled JS, and migrations are high-churn and high-complexity by nature. *Check:* read the top 10 paths aloud against the exclusion list; any file no human edits directly must move to the appendix.

3. **Complexity metric silently fails on part of the codebase.** The tool skips files it cannot parse, so those files get complexity 0 and vanish from the ranking. *Check:* compare the count of files the complexity tool reported against the count of non-excluded source files. If the gap exceeds 20%, stop per the rules.

4. **The ranking is statistically real but practically useless.** The top file is a 3,000-line legacy module everyone already knows about and nobody is allowed to touch. *Check:* step 10's read-through plus step 11's "name a specific first move." If you cannot name a move for the top 3, the map has not earned its deliverable — add an explicit note about constraints (ownership, freeze, planned deletion) rather than shipping advice no one can act on.

## License

MIT
