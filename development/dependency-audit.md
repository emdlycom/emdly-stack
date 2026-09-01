---
name: dependency-audit
owner: launifycorp
category: Development
description: Audits a project's dependencies and produces a prioritised upgrade plan: what is a security risk, what has drifted behind, and what is not used at all. It reports and recommends — it does not install,...
version: v1
license: MIT
updated: 2026-09-01
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/dependency-audit
raw: https://emdly.com/raw/launifycorp/dependency-audit.md
install: npx @emdly/cli add launifycorp/dependency-audit
---

# Dependency Audit

Audits a project's dependencies and produces a prioritised upgrade plan: what is a security risk, what has drifted behind, and what is not used at all. It reports and recommends — it does not install, upgrade, or edit a manifest unless the user explicitly asks for a specific change.

## When to use

- "audit my dependencies", "what packages should I upgrade", "are my deps safe"
- Before a release, a security review, or picking up a project that has sat untouched.
- After a vulnerability report lands and you need to know whether it actually affects this project.

Do not use it to upgrade a single named package — that is one command, not an audit.

## Inputs to establish first

Detect rather than ask wherever possible.

- **Project root** — default: the current working directory.
- **Ecosystem** — from the manifest and lockfile present:
  - JS/TS: `package.json` + `package-lock.json` / `yarn.lock` / `pnpm-lock.yaml` / `bun.lockb`
  - Python: `pyproject.toml`, `requirements*.txt`, `poetry.lock`, `uv.lock`, `Pipfile.lock`
  - Rust: `Cargo.toml` + `Cargo.lock`   Go: `go.mod` + `go.sum`
  - Ruby: `Gemfile` + `Gemfile.lock`    PHP: `composer.json` + `composer.lock`
  - Java: `pom.xml`, `build.gradle`
- **Monorepo** — if there are workspaces, ask whether to audit all packages or one, and report per package.
- If several ecosystems are present, audit each and keep the findings separate.

Always read the **lockfile** for what is actually installed, and the **manifest** for what is declared. The gap between them matters: a transitive vulnerability cannot be fixed by bumping the manifest entry.

## Steps

1. **Inventory.** Parse the lockfile into: package, installed version, declared range, direct or transitive, and which dependency group (prod / dev / peer / optional). A vulnerability in a dev-only package is not a production risk — never conflate the two.

2. **Security.** Run the ecosystem's own audit tool and treat its output as the source of truth:
   `npm audit --json`, `pnpm audit --json`, `yarn npm audit --json`, `pip-audit -f json`, `cargo audit --json`, `govulncheck ./...`, `bundle audit`, `composer audit --format=json`.
   If a tool is unavailable, say so and mark that section incomplete rather than guessing from memory. Model knowledge of CVEs is stale by definition — never invent an advisory ID, a CVSS score, or an affected range.
   For each advisory record: package, severity, advisory ID, affected range, first patched version, whether it is reachable from a direct dependency, and whether a fix exists at all.

3. **Staleness.** For each direct dependency compare installed against latest (`npm outdated --json`, `pip list --outdated --format=json`, `cargo outdated`, `go list -m -u all`). Classify: patch behind, minor behind, one major behind, several majors behind. Note anything unmaintained — no release in over two years, or a deprecation notice in the registry metadata.

4. **Unused and missing.** Cross-check declared dependencies against real imports in the source (`import` / `require` / `from x import` / `use`). Report as *likely* unused, and exclude the usual false positives: type packages, ESLint/Babel/PostCSS plugins loaded by config, build tools, test runners, anything referenced in a script, Dockerfile, or CI config. Also flag the reverse — packages imported in code but not declared.

5. **Prioritise** by risk against effort:

   | Tier | Contents |
   |---|---|
   | **1 — Fix now** | Production dependency, high/critical advisory, patch available within the current major |
   | **2 — Plan** | Advisory needing a major bump, or a production dependency several majors behind |
   | **3 — Housekeeping** | Dev-only advisories, minor drift, unused packages |
   | **Blocked** | Advisory with no patched version, or a fix that conflicts with another pin — say what blocks it |

   For every major-version bump, state that it may be breaking and point at the changelog or migration guide rather than asserting what changed.

6. **Present the report and stop.** Do not run installs, do not edit the manifest, do not regenerate the lockfile as a side effect of investigating. If the user then asks for changes, apply them one tier at a time, run the test suite after each, and report what broke.

## Rules

- Never invent CVE identifiers, severity scores, or patched versions. Every advisory must come from an audit tool's output.
- Never claim a project is secure. Say what was scanned, by which tool, and on what date; unscanned is not clean.
- Never run a command that writes — no `npm audit fix`, no `--force`, no lockfile regeneration — without explicit confirmation naming the package.
- Keep dev and production findings separate throughout.
- If the lockfile is missing, say that results are approximate: only ranges are known, not installed versions.
- Report registry or network failures plainly instead of silently returning a shorter list.

## Output format

```
Project: <name>   Ecosystem: <npm|pip|cargo|...>   Lockfile: <file>
Dependencies: <n> direct, <n> transitive   Scanned with: <tool> on <date>

TIER 1 — Fix now (<n>)
  <pkg> <installed> → <patched>   CRITICAL  <advisory-id>   prod, direct
    <one line: what it is and why it matters here>

TIER 2 — Plan (<n>)
  <pkg> <installed> → <latest>    major bump, breaking changes likely
    <blocker or migration note>

TIER 3 — Housekeeping (<n>)
  <pkg>   3 minors behind   dev only
  <pkg>   declared but no imports found — likely unused

BLOCKED (<n>)
  <pkg>   <advisory-id>   no patched version published

Not scanned: <anything skipped, and why>
```

Close with the three actions worth doing first, in order, and nothing more. No changes have been made.

## License

MIT
