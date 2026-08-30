---
name: dependency-upgrade-planner
owner: kernelpanic
category: Development
description: Reads a lockfile diff or an outdated report and plans the upgrade — order, breaking changes to read, and the smallest safe steps.
version: v3
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/kernelpanic/dependency-upgrade-planner
raw: https://emdly.com/raw/kernelpanic/dependency-upgrade-planner.md
install: npx @emdly/cli add kernelpanic/dependency-upgrade-planner
---

# Dependency upgrade planner

Upgrades fail when they are done as one big bump. This skill turns "47 packages outdated" into an ordered list of small, reversible steps, each with a named proof that it worked. The plan is for the engineer who will run it alone on a Thursday afternoon: every step says what to change, what to read before changing it, and what must pass before the next step starts.

## When to use
- After `composer outdated`, `npm outdated`, `pip list --outdated`, `bundle outdated` or a Dependabot summary.
- Before a framework major, to find out how many steps it actually is.
- When a security advisory forces one package forward and you need to know what it drags with it.
- When an upgrade has already stalled and you need the remainder re-ordered around what is now green.

## Input
Required:
- The outdated report, with name, current, wanted and latest for each package.
- The lockfile, for the resolved transitive versions and the peer constraints.
- The CI status now, before anything moves. A plan built on an already-red build is not a plan.
- Whether the project has an integration test suite, and roughly what it covers.

**Required for step 3, and the reason step 3 exists.** For every package you will classify as a major, supply at least one of:
- the package's `UPGRADE.md`, `UPGRADING.md` or `CHANGELOG.md` at the target version;
- its GitHub releases page for every version between current and target;
- the vendor's published upgrade guide (URL);
- the diff of its public API between the two tags.

And, to turn a breaking change into a file list:
- **read access to the codebase**, so the removed and renamed APIs can be grepped for.

If a package has none of these, it is not planned as a major. See "No upgrade guide" under Edge cases. If there is no code access at all, the method collapses — say so and stop rather than shipping a version-number sort.

Optional: the runtime version (PHP/Node/Python) and its support window, the deploy cadence, and any package the team has already decided to drop.

## Process
1. **Sort by blast radius.** Runtime first, then the framework, then packages the code imports directly, then transitive-only. Report the four buckets with counts.
2. **Classify each bump** by the semver distance between current and latest, not by how the report labels it.
   - **Patch**: batch them. One step, one proof.
   - **Minor**: batch per ecosystem. Read release notes for every package the code imports directly.
   - **Major**: one per step, always, and always alone.
3. **Read, don't guess.** For each major, open the source you were given in Input, and for each breaking change ask one question: does this codebase touch it? Quote the breaking change in the words of the guide, name the section it came from, then grep for the removed or renamed symbol and list the files that match. A breaking change with no file list is either "not used here" — say that — or unproven, which is a stop (below). Never write "may require changes".
4. **Order the majors** so each step leaves the build green on its own. A package whose new version requires a newer framework goes after the framework. Where two packages require each other's new versions, they are one step, and say so.
5. **Define each step's proof.** Name the exact command or click-path that must pass: a test suite, a specific test file, an artisan/rake/npm command, a manual flow with its steps. "Smoke test" is not a proof. Every step also names its rollback: the lockfile revert, plus any migration or config that must be undone.
6. **Report what you held back**, with the reason and the issue link when one exists.

## Rules
- Never bundle a major with anything else — not a patch batch, not a config cleanup, not a rename.
- Version numbers are not evidence. Minor releases have broken people. Read the notes for anything the code imports directly.
- Every breaking change in the plan carries three things: the quote, its source (file and section, or URL), and the file list from the grep. Missing any of the three, it does not go in the plan.
- If the project has no tests, say so in the first line of the plan and propose exactly **three** smoke checks that stand in for them: one that exercises the framework's boot path, one that exercises the app's primary write, one that exercises its primary read. Three is the floor, not a target. [house rule; three is the smallest set that covers boot, write and read, which is where framework majors break first.]
- Pin what you cannot upgrade yet, and say why — abandoned package, peer conflict, paid version behind the wall — with the issue link when there is one.
- Cap a patch batch at **40 packages**. Above that, split by ecosystem or by top-level directory, because a red suite after 90 bumps costs more to bisect than the second step costs to run. [judgment, anchored on bisect cost, not a vendor number.]
- Report the number of steps, and the number of packages the plan does not move.

> Thresholds above are defaults; report the thresholds you used.

## Output format

```
## Inputs
Report: composer outdated (47 packages). Lockfile: yes. CI: green (3 jobs).
Integration suite: yes, 218 tests, covers billing and auth; no coverage of the queue.
Guides read: UPGRADE-13.0.md (laravel/framework), GitHub releases 3.0.0-3.2.1
(intervention/image). Code access: yes.

## Plan (6 steps)

1. **Patch batch** — 31 packages, no code touched.
   Proof: full test suite (218 tests).
   Rollback: `git checkout composer.lock && composer install`.

2. **symfony/* 7.3 -> 7.4 (minor)** — 6 packages, batched.
   Read: "Passing a name as first argument of Command's constructor is deprecated;
   use the #[AsCommand] attribute" (UPGRADE-7.4.md, Console section).
   Files: app/Console/Commands/SyncInvoices.php:14, app/Console/Commands/PruneTokens.php:11.
   Proof: `php artisan list` exits 0 and lists both commands, then full suite.
   Rollback: lockfile revert.

3. **laravel/framework 12 -> 13 (major)** — alone.
   Breaking, 3 items, all confirmed present in this codebase:
   a. "The `Container::call()` method no longer resolves string callables of the form
      `Class@method`." (UPGRADE-13.0.md, Container) — 4 files:
      app/Jobs/DispatchWebhook.php:38, app/Http/Middleware/Impersonate.php:22,
      routes/console.php:9, app/Providers/RouteServiceProvider.php:61.
   b. "The `$dates` property on Eloquent models has been removed; use `casts()`."
      (UPGRADE-13.0.md, Eloquent) — 9 models under app/Models/, listed in
      docs/upgrade-13-dates.txt (grep: `protected \$dates`).
   c. "String helpers `Str::studly()` and `Str::camel()` no longer cache results."
      (UPGRADE-13.0.md, Support) — 1 hot path: app/Support/Serializer.php:44,
      called per row in the export job. Benchmark before and after the bump.
   Not used here, from the same guide: the Blade component alias removal, the
   `Bus::dispatchNow` removal, the Redis `predis` default change.
   Proof: full suite, then a manual login -> create invoice -> export CSV pass on staging,
   then the export job on 10k rows with the timing from item (c) recorded.
   Rollback: lockfile revert plus `php artisan config:clear`; no migration in this step.

4. **spatie/laravel-permission 6 -> 7 (major)** — after step 3; 7.x requires Laravel 13.
   Breaking, 1 item: "The `role` middleware alias is no longer registered automatically."
   (CHANGELOG.md, 7.0.0) — 3 files: routes/web.php:22, routes/api.php:47,
   app/Http/Kernel.php:58.
   Proof: suite, plus an authenticated request to /admin as a non-admin returns 403.
   Rollback: lockfile revert plus restoring the alias line in Kernel.php.

5. **Node minor batch** — 12 packages from package.json, none imported by app code
   (build tooling only).
   Proof: `npm run build` produces a bundle within 5% of the current byte size.
   Rollback: package-lock.json revert.

6. **guzzlehttp/guzzle 7.8 -> 7.9 (minor)** — held to last because it is the only
   package the queue touches and the queue has no test coverage.
   Read: no breaking changes listed in releases 7.9.0-7.9.2.
   Proof: send one webhook to the staging receiver and confirm a 2xx in the log.
   Rollback: lockfile revert.

## Held back (3 packages)
- intervention/image 2.7 -> 3.2 — 3.x renames the facade and changes the driver
  contract; touches 14 files. Separate PR, not this plan.
- doctrine/annotations 1.14 — abandoned upstream (composer marks it abandoned, suggests
  doctrine/attributes). Pin at 1.14. Removal is a code change, not an upgrade.
- vendor/legacy-pdf 2.1 -> 3.0 — no changelog, no releases page, no public repo.
  Unprovable; see the stop below.

## Not moved
16 of 47 packages stay where they are: 3 held back above, 13 transitive-only that the
patch batch already resolves.
```

## Edge cases
- **No upgrade guide, changelog or release notes for a major.** The central step cannot run. Do not plan the bump from the version number. Move the package to "Held back" with the reason "unprovable — no published guide", and propose the one thing that does work: diff the two tags' public API yourself, or pin and open an issue asking upstream. Say which you did.
- **No code access.** Steps 3 and 5 collapse: you cannot produce file lists and you cannot name a proof. Say the method does not work without the repo, and report only the classification and the ordering, labelled "ordering only, no impact analysis". Do not present that as a plan.
- **No lockfile.** Transitive versions are unknown, so peer conflicts cannot be predicted. Plan the direct dependencies only and say the transitive graph was not read.
- **No outdated report, or a malformed one** (missing the current or latest column). Ask for `composer outdated --direct --format=json` or the equivalent. Do not scrape versions out of the manifest — the manifest holds constraints, not resolved versions.
- **Empty report: nothing is outdated.** Say so in one line, and check one thing before closing: whether any pinned package is behind because of a constraint. Report those, or "none".
- **A very large report** (over 150 packages). Do not plan every line. Plan the runtime, the framework, and every direct dependency; report the transitive tail as a count with the ecosystems named, and say it was not individually reviewed.
- **CI is already red.** Stop before planning. The proofs in every step depend on a green baseline. Report what is failing and say the plan starts once it is green.
- **The project has no tests and no staging environment.** The three smoke checks have nowhere to run. See the stop below — this is not a case to work around.
- **Two majors that require each other.** They are one step. Say so, say why, and accept that this step is bigger; name the extra proof that compensates.
- **The runtime itself is out of support** (PHP, Node or Python past end of life). Plan the runtime first, as step 1, before any package. Nothing else in the plan is safe on an unsupported runtime.
- **A package is behind because the team pinned it deliberately.** Do not unpin it. Report the pin, its stated reason if the input carries one, and "reason not recorded" if it does not.

## Stop and hand back
This skill plans changes to production dependencies. It writes plans; it never runs an upgrade, never edits a lockfile, and never opens a PR. Stop and hand to a named person on any of these:

- **A step you cannot prove.** No guide, no releases page, no API diff, or no code access to produce the file list. Do not write the step. Move the package to "Held back", say what evidence is missing, and name the engineer who owns that dependency as the decider.
- **A package is out of security support** — end of life, abandoned upstream, or carrying an unpatched advisory at the current pin. This stops being an upgrade plan and becomes a risk decision about running unsupported code in production. Report the package, the advisory id if you have one, and route to the security owner before the plan proceeds.
- **The project has neither a test suite nor anywhere to run the three smoke checks.** There is no proof available for any step, so every step is unverifiable. Say so plainly, hand the plan back to the tech lead, and propose the smoke checks as work to do *before* the first bump, not as part of it.
- **A framework or runtime major on a system that is live.** Publish the plan, then require a named owner to sign off on the step order and the rollback before step 1 runs. State the maintenance window question; do not answer it.
- **A bump that forces a data migration** — a change to an ORM, a queue backend, a serializer, or a cache format. Name what happens to data already written in the old format, and route to whoever owns that store.
- **A licence change between the two versions** (for example a move to BSL, SSPL or a commercial tier). Stop and route to whoever owns licensing. Do not evaluate the licence yourself.

## License
MIT
