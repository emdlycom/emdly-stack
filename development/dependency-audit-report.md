---
name: dependency-audit-report
owner: launifycorp
category: Development
description: This skill produces a prioritized, evidencebacked report of every direct and transitive dependency in a legacy codebase that is outdated, carries a known vulnerability, or is abandoned — each with a c...
version: v1
license: MIT
updated: 2026-09-18
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/dependency-audit-report
raw: https://emdly.com/raw/launifycorp/dependency-audit-report.md
install: npx @emdly/cli add launifycorp/dependency-audit-report
---

# Dependency Audit Report

This skill produces a prioritized, evidence-backed report of every direct and transitive dependency in a legacy codebase that is outdated, carries a known vulnerability, or is abandoned — each with a concrete upgrade path, breaking-change summary, and effort estimate. You own the report artifact and the accuracy of every claim in it. You do not own performing the upgrades.

## When to use

- A legacy service is being picked up again after months or years of no maintenance and nobody knows what is safe to touch.
- A security or compliance review demands a dependency inventory with CVE status and remediation plans.
- A framework or runtime end-of-life date is approaching (e.g. Node 16, Python 3.7, Rails 5) and someone must scope the migration.
- A build broke or a package registry dropped a version, and you need to know how much of the tree is fragile.
- Before a re-platform, rewrite-vs-refactor decision, or vendor handover, where dependency debt is an input to the estimate.

Do not use when:

- The task is to actually perform the upgrades and land PRs — that is a migration job; this skill stops at the recommendation.
- You lack read access to the dependency manifests and lockfiles; a report built from guessed versions is worse than no report.

## Inputs

Before starting, collect:

1. **Repository access** — full checkout, including branch to audit (default: the deployed branch, not `main`, if they differ).
2. **Manifest and lockfile pairs** for every package ecosystem in the repo: `package.json`/`package-lock.json`/`yarn.lock`/`pnpm-lock.yaml`, `requirements.txt`/`Pipfile.lock`/`poetry.lock`, `Gemfile`/`Gemfile.lock`, `pom.xml`/`build.gradle`, `go.mod`/`go.sum`, `composer.json`/`composer.lock`, `*.csproj`/`packages.lock.json`.
3. **Runtime versions in production** — language runtime, OS base image, and any container tags. Get these from Dockerfiles, CI config, or deploy manifests, not from someone's memory.
4. **Deployment and support context** — is this system internet-facing, does it process personal or payment data, what is the expected remaining lifetime.
5. **Test coverage signal** — does a test suite exist, does it pass today, roughly what percentage of code it covers.

If any are missing, ask for them explicitly and name the file you need:

- No lockfile → ask: "Is there a lockfile committed anywhere, or is this deployed from resolved-at-build-time manifests?" If genuinely absent, say so in the report and mark all transitive findings `UNVERIFIED`.
- No production runtime version → ask for the Dockerfile, CI workflow, or a `--version` output from a running instance. Do not assume latest.
- No answer on internet exposure → assume internet-facing and note the assumption; this raises severity ceilings, which is the safe direction.
- Test suite status unknown → run it once yourself if the repo builds; if it does not build, record "unbuildable at audit time" and raise every effort estimate one band.

## Method

1. **Inventory ecosystems.** Walk the repo and list every manifest file found, including ones in subdirectories, vendored folders, and CI-only tooling. If more than one ecosystem is present, audit each separately and never merge their version numbers into one table. If a vendored `node_modules`, `vendor/`, or checked-in JAR directory exists, flag it — those bypass lockfiles and must be inventoried by reading package metadata directly.
2. **Resolve the actual installed tree.** Prefer the lockfile over the manifest; the manifest states intent, the lockfile states reality. Produce a flat list of `name@exact-version` for direct and transitive dependencies. If lockfile and manifest disagree (manifest allows `^4.0.0`, lockfile pins `4.0.1`), report the lockfile version and note the drift.
3. **Pull current-version data.** For each package, record the latest stable release and the latest release within the current major. Source this from the registry, not from a language model's memory. If you cannot reach a registry, stop and say so rather than estimating — an invented "latest version" invalidates the whole report.
4. **Scan for vulnerabilities.** Run the ecosystem's native tool (`npm audit --json`, `pip-audit`, `bundle audit`, `govulncheck`, `mvn dependency-check`, `composer audit`) and cross-check against the OSV database. For each finding, record CVE/GHSA ID, CVSS score, affected version range, fixed version, and whether the vulnerable code path is actually reachable from this application. If reachability is undetermined, mark it `REACHABILITY UNKNOWN` and treat it as reachable for prioritization.
5. **Classify abandonment.** Mark a package `ABANDONED` if two or more hold: no release in 24+ months, repository archived or deleted, open issue count rising with zero maintainer replies in 12 months, or an explicit deprecation notice on the registry. Mark `AT RISK` if exactly one holds. For every abandoned package, identify at least one maintained successor or fork, or state explicitly that none exists and removal is the only path.
6. **Determine the upgrade path per package.** Choose one: `PATCH` (same minor, no API change), `MINOR` (additive only), `MAJOR` (breaking — you must read the changelog and list the specific breaking changes that touch this codebase), `REPLACE` (abandoned, swap to named alternative), `REMOVE` (unused — verify by grepping for imports before asserting this), or `BLOCKED` (upgrade requires a runtime or peer-dependency bump first). For `BLOCKED`, name the blocker and put it earlier in the sequence.
7. **Grep for actual usage before recommending anything invasive.** For every `MAJOR`, `REPLACE`, or `REMOVE`, count import sites and list the files. A three-import change and a three-hundred-import change are not the same recommendation, and the report must distinguish them.
8. **Assign severity using a fixed rule.** `CRITICAL`: known-exploited or CVSS ≥ 9.0 on a reachable, internet-facing path. `HIGH`: CVSS 7.0–8.9 reachable, or a runtime hitting EOL within 6 months. `MEDIUM`: CVSS 4.0–6.9, or abandoned package on a critical path. `LOW`: everything else, including cosmetic version drift. Do not adjust severity for how hard the fix is; effort is a separate column.
9. **Estimate effort in bands, not hours.** `S` = under half a day, mechanical. `M` = 1–3 days, requires code changes and test updates. `L` = over 3 days, or touches architecture, or cannot be validated because tests do not cover it. Any change to an untested module is minimum `M`.
10. **Sequence the work.** Order remediation so blockers resolve first, then CRITICAL/HIGH security items, then runtime EOL, then abandonment replacements, then version hygiene. Group changes that must ship together (peer-dependency sets, framework plus its plugins) into single numbered waves. Do not list more than 5 waves; if you need more, the project needs a migration plan, not an audit, and you should say that.
11. **Write the report** using the template below, then re-verify every version number and CVE ID against your source data before delivering. Delete rows you cannot substantiate.

## Rules

- Never state a version number, CVE ID, CVSS score, or release date you did not read from a tool output or registry response during this audit. No recalled values.
- Never recommend "upgrade everything to latest." Legacy projects break on that instruction. Every recommendation must be per-package with a stated path.
- Never mark a package `REMOVE` without showing the grep result that proves zero usage, including dynamic imports, config-file references, and CI scripts.
- Never downgrade severity because remediation is expensive or because the team "will not do it." Severity describes risk; effort describes cost; they stay in separate columns.
- Never audit `main` when production runs a different branch or tag. Confirm the branch and record it in the report header.
- If a registry, advisory database, or the build itself is unreachable, mark the affected section `INCOMPLETE — <reason>` and continue with the rest. Do not silently omit it.
- Transitive vulnerabilities must name the direct parent that pulls them in; a finding a team cannot act on is not a finding.
- Cap the detail tables at the 40 highest-severity findings. Summarize the remainder as counts by severity and attach the full machine-readable list separately.
- Do not propose upgrades that require a runtime version unavailable in the project's deployment target. Check the base image and CI matrix first.
- Where the safest action is to freeze and isolate rather than upgrade (unmaintained package, no successor, low exposure), say that explicitly instead of forcing an upgrade path.

## Output format

````markdown
# Dependency Audit Report — <project name>

**Repository:** <repo> | **Branch/tag audited:** <ref> | **Commit:** <sha>
**Date:** <YYYY-MM-DD> | **Ecosystems:** <npm, pip, ...>
**Exposure assumption:** <internet-facing | internal-only> (<source of this claim>)
**Build status at audit time:** <builds clean | builds with warnings | does not build>
**Test suite:** <passing/failing/absent>, <coverage % or "unknown">

## 1. Summary

<3-5 sentences: total dependency count, how many findings by severity, the single
most urgent item, and whether the project is upgradable in place or needs a
migration plan.>

| Metric | Count |
|---|---|
| Direct dependencies | <n> |
| Transitive dependencies | <n> |
| Critical findings | <n> |
| High findings | <n> |
| Medium findings | <n> |
| Low findings | <n> |
| Abandoned packages | <n> |
| Runtime EOL risks | <n> |

## 2. Runtime and platform status

| Component | In use | Latest supported | EOL date | Status |
|---|---|---|---|---|
| <Node/Python/Ruby/JVM> | <ver> | <ver> | <date> | <OK / EOL in Nm / EOL> |
| <base image> | <tag> | <tag> | <date> | <status> |

<One paragraph on what the runtime status blocks. If nothing is blocked, say so.>

## 3. Vulnerability findings

| Severity | Package | Installed | Fixed in | CVE/GHSA | CVSS | Direct/Transitive (parent) | Reachable | Effort |
|---|---|---|---|---|---|---|---|---|
| CRITICAL | <pkg> | <ver> | <ver> | <id> | <score> | Transitive (<parent>) | Yes | S |

<For each CRITICAL and HIGH row, add a 2-3 sentence note below the table: what
the vulnerability allows, where in this codebase it is reachable, and any
mitigation available short of upgrading.>

## 4. Abandonment findings

| Package | Installed | Last release | Signal | Successor | Import sites | Effort |
|---|---|---|---|---|---|---|
| <pkg> | <ver> | <YYYY-MM> | <archived / no maintainer response / deprecated> | <name or NONE> | <n> (<files>) | <S/M/L> |

## 5. Version drift

| Package | Installed | Latest in major | Latest stable | Path | Breaking changes affecting us | Effort |
|---|---|---|---|---|---|---|
| <pkg> | <ver> | <ver> | <ver> | MAJOR | <bullet list, or "none identified"> | <S/M/L> |

<Remaining low-severity drift: <n> packages, all PATCH/MINOR. Full list in appendix.>

## 6. Remediation sequence

### Wave 1 — <name> (unblocks: <what>)
- [ ] <action> — <package(s)> — Effort: <S/M/L> — Risk if deferred: <one line>
- [ ] <action> — ...
**Exit criteria:** <what must be true before Wave 2 starts>

### Wave 2 — <name>
- [ ] ...
**Exit criteria:** <...>

<Repeat up to Wave 5.>

## 7. Do not upgrade

| Package | Reason to hold | Alternative control |
|---|---|---|
| <pkg> | <e.g. no maintained successor, forked internally, pinned by vendor contract> | <isolate / WAF rule / remove feature> |

## 8. Gaps and assumptions

- <Anything marked UNVERIFIED, INCOMPLETE, or REACHABILITY UNKNOWN, and what
  would be needed to close it.>
- <Every assumption made in the absence of an input, with its direction of bias.>

## Appendix A — Full dependency inventory
<Attached as <filename>.json/csv, or inline if under 50 entries.>

## Appendix B — Tool output
<Commands run, tool versions, raw output locations.>
````

## Failure modes

1. **Hallucinated version numbers and CVE IDs.** The report looks authoritative and is fabricated in places, which destroys trust in all of it. *Check:* before delivery, take a random sample of five rows across the tables and re-verify each against the tool output or registry response saved in Appendix B. If any one fails, re-verify the entire table it came from.
2. **Auditing the manifest instead of the deployed tree.** You report `^2.1.0` as "2.1.0, current" while production has resolved to a vulnerable 2.1.0 build or a different branch entirely. *Check:* confirm the commit SHA in the report header matches what is deployed, and that every version in the tables came from a lockfile or installed-package metadata, not a semver range.
3. **Recommending an upgrade the runtime cannot support.** You tell the team to move to a package version requiring Node 20 while the base image pins Node 14, producing a plan that fails at step one. *Check:* for every `MAJOR` and `REPLACE` recommendation, confirm the target version's declared engine/runtime constraint is satisfied by the platform row in section 2. Any that are not must become `BLOCKED` with the runtime bump sequenced first.
4. **Effort estimates ignoring test coverage.** Mechanical-looking upgrades get rated `S`, then take a week because nothing validates the change. *Check:* cross-reference each `S` estimate against the files listed in its import sites; if those files have no corresponding tests, the estimate is `M` or higher.
5. **Findings with no actionable owner.** Transitive vulnerabilities listed without their parent package leave the team unable to act, and the report gets shelved. *Check:* every row where Direct/Transitive is "Transitive" must name a parent, and that parent must appear somewhere in the remediation sequence.

## License

MIT
