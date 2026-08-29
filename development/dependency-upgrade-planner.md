---
name: dependency-upgrade-planner
owner: kernelpanic
category: Development
description: Reads a lockfile diff or an outdated report and plans the upgrade — order, breaking changes to read, and the smallest safe steps.
version: v2
license: MIT
updated: 2026-08-18
recommended: false
security_checked: true
url: https://emdly.com/skills/kernelpanic/dependency-upgrade-planner
raw: https://emdly.com/raw/kernelpanic/dependency-upgrade-planner.md
install: npx @emdly/cli add kernelpanic/dependency-upgrade-planner
---

# Dependency upgrade planner

Upgrades fail when they are done as one big bump. This skill turns "47 packages outdated" into an ordered list of small, reversible steps.

## When to use
- After `composer outdated`, `npm outdated`, `pip list --outdated` or a Dependabot summary.
- Before a framework major.

## Input
The outdated report (name, current, wanted, latest), the lockfile, the CI status, and whether the project has an integration test suite.

## Process
1. **Sort by blast radius.** Framework and runtime first, then packages the code imports directly, then transitive-only.
2. **Classify each bump.** Patch: batch them. Minor: batch per ecosystem, read release notes for the ones the code imports. Major: one at a time, always.
3. **Read, don't guess.** For every major, find the upgrade guide or changelog and quote the breaking changes that touch this codebase — grep for the removed APIs and list the files.
4. **Order the majors** so that each step leaves the build green: a package that requires a newer framework goes after the framework.
5. **Define the step's proof**: which tests or smoke checks must pass before the next step.

## Rules
- Never bundle a major with anything else.
- Version numbers are not evidence. "Minor" releases have broken people; read the notes for anything the code imports.
- If the project has no tests, say so first and propose the three smoke checks that stand in for them.
- Pin what you cannot upgrade yet, and say why (an abandoned package, a peer conflict) — with the issue link when there is one.

## Output format
```
## Plan (6 steps)
1. Patch batch — 31 packages, no code touched. Proof: full test suite.
2. symfony/* 7.3 → 7.4 (minor) — reads: Console `Command::$defaultName` deprecation, used in app/Console/*. Proof: `php artisan list` + suite.
3. laravel/framework 12 → 13 (major) — breaking: … (3 items, files listed). Proof: suite + manual login flow.
...
## Held back
- intervention/image 2.x — 3.x renames the facade; touches 14 files. Separate PR.
```

## License
MIT
