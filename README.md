# emdly stack

Every published skill on [emdly](https://emdly.com) — Markdown playbooks for AI agents, sorted by category. Each file carries a front-matter header (owner, version, license, URLs). Install any of them with `npx @emdly/cli add owner/skill`, fetch the raw file, or connect the catalog over MCP (`claude mcp add --transport http emdly https://emdly.com/mcp`).

This repository is generated from the catalog; edits happen on emdly (submit a skill, it goes through review). 39 skills · updated 2026-08-31.

## Categories

- [Code review](#code-review) (2)
- [Copywriting](#copywriting) (2)
- [Data & analytics](#data-analytics) (2)
- [Design](#design) (1)
- [Development](#development) (5)
- [Docs & changelogs](#docs-changelogs) (2)
- [Ecommerce](#ecommerce) (5)
- [Email & outreach](#email-outreach) (2)
- [Finance](#finance) (1)
- [Game development](#game-development) (2)
- [Hiring](#hiring) (1)
- [Learning](#learning) (1)
- [Product](#product) (1)
- [Research](#research) (1)
- [SEO](#seo) (3)
- [Security](#security) (2)
- [Skills](#skills) (1)
- [Support](#support) (1)
- [Team ops](#team-ops) (1)
- [Ticket ops](#ticket-ops) (3)

## Code review

| skill | description | version | |
|---|---|---|---|
| [kernelpanic/commit-message-editor](code-review/commit-message-editor.md) | Rewrites a commit message from the diff — imperative subject under 72 characters, a body that explains why, and a footer that links the issue. | v4 | 🛡 |
| [kernelpanic/pr-review-ritual](code-review/pr-review-ritual.md) | A reviewing discipline for agents — read the diff twice, test the edge cases, comment on intent, never nitpick formatting a linter owns. | v5 | ★ 🛡 |

## Copywriting

| skill | description | version | |
|---|---|---|---|
| [tonecheck/brand-voice-guard](copywriting/brand-voice-guard.md) | Holds any draft against your voice guide — banned words, sentence rhythm, claims policy — and returns a marked-up pass, not a rewrite. | v6 | 🛡 |
| [tonecheck/landing-page-copy-editor](copywriting/landing-page-copy-editor.md) | Edits a landing page section by section — one promise above the fold, proof before features, and a CTA that says what happens next. | v3 | 🛡 |

## Data & analytics

| skill | description | version | |
|---|---|---|---|
| [querydeck/dashboard-metric-definer](data-analytics/dashboard-metric-definer.md) | Turns a vague metric request ("active users", "churn") into a precise definition with grain, window, filters and the SQL skeleton — before anyone builds the chart. | v4 | 🛡 |
| [querydeck/sql-query-explainer](data-analytics/sql-query-explainer.md) | Explains what a query actually does — joins, filters, gotchas — in plain language, then flags the index it wishes existed. | v5 | ★ 🛡 |

## Design

| skill | description | version | |
|---|---|---|---|
| [launifycorp/design-review](design/design-review.md) | A design review is not a list of everything wrong with a page. Anyone can produce forty observations. The work is deciding which three matter, and showing why. This skill is built around that: evidenc... | v1 | ★ 🛡 |

## Development

| skill | description | version | |
|---|---|---|---|
| [kernelpanic/dependency-upgrade-planner](development/dependency-upgrade-planner.md) | Reads a lockfile diff or an outdated report and plans the upgrade — order, breaking changes to read, and the smallest safe steps. | v4 | 🛡 |
| [sevzero/incident-postmortem](development/incident-postmortem.md) | Drafts blameless postmortems from a timeline and a channel export — impact, contributing factors, and actions with owners. | v4 | ★ 🛡 |
| [sevzero/k8s-incident-triage](development/k8s-incident-triage.md) | Given kubectl output and alerts, ranks probable causes and proposes the next safe diagnostic step — read-only commands only, never a mutation without a human. | v4 |  |
| [launifycorp/linux-web-server-triage](development/linux-web-server-triage.md) | Read-only triage for a single Linux box serving a site over nginx and PHP-FPM. Separate ladders for "down" and "slow", a verbatim read-only command allowlist, the 502/504/499 distinction by errno, PHP-FPM pool saturation versus memory_limit versus OOM kill, certbot renewal traps and rate limits, load read against core count, and inode exhaustion. Proposes commands; never executes, never mitigates. | v3 | 🛡 |
| [launifycorp/n8n-api](development/n8n-api.md) | Drive an n8n instance through its public REST API — audit workflows, diagnose failed executions, edit safely. Built around the two things that bite: a workflow PUT is a full replacement, and publishing starts a workflow firing against production. Read first, keep the baseline, never publish on your own authority. | v2 | 🛡 |

## Docs & changelogs

| skill | description | version | |
|---|---|---|---|
| [shiplog/api-reference-writer](docs-changelogs/api-reference-writer.md) | Writes an endpoint's reference page from its route, request validation and response shape — every field, every error, one working example. | v3 | 🛡 |
| [shiplog/changelog-composer](docs-changelogs/changelog-composer.md) | Turns a week of merged PRs into a changelog humans read — grouped by impact, written for users, breaking changes on top. | v4 | 🛡 |

## Ecommerce

| skill | description | version | |
|---|---|---|---|
| [shopmetric/ga4-funnel-analyst](ecommerce/ga4-funnel-analyst.md) | Reads a GA4 export and finds where the checkout funnel leaks — drop-off by step, device and source — with the one fix to try first. | v5 | ★ 🛡 |
| [cartlift/product-feed-optimizer](ecommerce/product-feed-optimizer.md) | Rewrites product titles and descriptions for Google Shopping feeds — attributes first, brand rules kept, no keyword stuffing. | v5 | 🛡 |
| [launifycorp/product-translator](ecommerce/product-translator.md) | Localises e-shop product records into a new market: titles, descriptions, attributes, categories and slugs. Every field leaves with one of four verdicts — translated, mapped, kept, formatted — so identifiers survive, categories map to numeric IDs instead of being translated, and no size or unit is silently converted. | v1 | 🛡 |
| [shopmetric/search-console-auditor](ecommerce/search-console-auditor.md) | Audits Google Search Console data — pages losing clicks, queries with impressions but no CTR, and product pages cannibalizing each other. | v5 | 🛡 |
| [launifycorp/shopify-store-audit](ecommerce/shopify-store-audit.md) | Pre-launch readiness audit of a Shopify store from its URL alone, no admin access. Reads what the storefront publishes about itself — theme, currency, catalogue JSON, and the fixed policy URLs that either resolve or do not — and reports what a first customer would hit. Never completes a checkout, never judges whether a policy is legally adequate. | v1 | 🛡 |

## Email & outreach

| skill | description | version | |
|---|---|---|---|
| [cartlift/abandoned-cart-sequence](email-outreach/abandoned-cart-sequence.md) | Drafts a three-mail abandoned-cart sequence from the cart contents and the store's voice — useful, specific, and honest about discounts. | v4 |  |
| [outboundlab/cold-email-first-touch](email-outreach/cold-email-first-touch.md) | Writes first-touch e-mails that sound like a person — one observation, one sentence of value, one soft ask. Kills every buzzword on sight. | v4 | 🛡 |

## Finance

| skill | description | version | |
|---|---|---|---|
| [ledgerline/burn-rate-analyst](finance/burn-rate-analyst.md) | Reads a monthly P&L export and reports runway, burn trends and the three line items moving fastest — with the math shown. | v4 | 🛡 |

## Game development

| skill | description | version | |
|---|---|---|---|
| [pixelforge/game-balance-reviewer](game-development/game-balance-reviewer.md) | Reads stat tables and patch notes, then flags outliers — dominant strategies, dead perks, and curves that punish new players. | v4 | 🛡 |
| [pixelforge/playtest-feedback-triage](game-development/playtest-feedback-triage.md) | Sorts raw playtest notes into balance, UX, bugs and feel — ranked by how many testers hit it, with repro steps pulled from context. | v5 | ★ 🛡 |

## Hiring

| skill | description | version | |
|---|---|---|---|
| [talentloop/take-home-grader](hiring/take-home-grader.md) | Grades take-home submissions against a rubric you define, blind to names, and writes feedback the candidate can actually use. | v4 | 🛡 |

## Learning

| skill | description | version | |
|---|---|---|---|
| [launifycorp/grill-me](learning/grill-me.md) | Puts you under hard questioning before someone else does — one question at a time, following the weakest thing you just said, never revealing the answer before you commit. Ends with a debrief separating what you established, what you asserted without support, what you did not know, and what you knew but could not say. | v1 | 🛡 |

## Product

| skill | description | version | |
|---|---|---|---|
| [deckhand/prd-sharpener](product/prd-sharpener.md) | Turns a draft PRD into one a team can build from — problem before solution, success metric with a number, explicit non-goals, and open questions with owners. | v4 | 🛡 |

## Research

| skill | description | version | |
|---|---|---|---|
| [fieldnotes/user-interview-distiller](research/user-interview-distiller.md) | Distills interview transcripts into claims with evidence — what was said, how often, and what it contradicts in your assumptions. | v5 | 🛡 |

## SEO

| skill | description | version | |
|---|---|---|---|
| [launifycorp/geo-audit](seo/geo-audit.md) | Generative engine optimisation audit from one URL: which AI crawlers are permitted and what each operator says that means, what a non-rendering fetch actually receives, whether any passage is self-contained enough to be lifted, and whether anything on the page could only have come from there. Reports configuration, never a visibility score, because none exists. | v1 |  |
| [launifycorp/seo-audit](seo/seo-audit.md) | Technical SEO audit that starts from one URL and works outward, ordered by consequence rather than by ease of checking: resolve, crawlability, indexability, rendering, canonicalisation, on-page, structured data, hreflang, speed. Every threshold is quoted from Google or the sitemaps protocol, or labelled as the skill's own judgment. | v1 | 🛡 |
| [rankcraft/seo-brief-builder](seo/seo-brief-builder.md) | Builds content briefs from a keyword — search intent, outline, entities to cover, and the questions the top ten never answer. | v3 | 🛡 |

## Security

| skill | description | version | |
|---|---|---|---|
| [nullptr/dependency-cve-triage](security/dependency-cve-triage.md) | Triages a vulnerability report (Dependabot, npm audit, composer audit) — reachable or not, exploitable in this app or not, and what to do this week versus later. | v4 | 🛡 |
| [nullptr/threat-model-sketch](security/threat-model-sketch.md) | Sketches a threat model for a feature from its description and data flow — assets, entry points, top threats by STRIDE, and the mitigation that exists or is missing. | v4 | 🛡 |

## Skills

| skill | description | version | |
|---|---|---|---|
| [promptsmith/skill-author-guide](skills/skill-author-guide.md) | How to write an emdly skill that passes review and actually works in an agent — structure, rules that bind, output contracts, and the mistakes reviewers bounce. | v5 | ★ 🛡 |

## Support

| skill | description | version | |
|---|---|---|---|
| [helpdeskly/support-reply-drafter](support/support-reply-drafter.md) | Drafts a reply to a support ticket from the conversation and the help center — answers the actual question first, promises only what the policy allows, and escalates when it should. | v3 | 🛡 |

## Team ops

| skill | description | version | |
|---|---|---|---|
| [opsmith/standup-synthesizer](team-ops/standup-synthesizer.md) | Reads yesterday's commits, tickets and threads, and writes each person's standup draft — blockers first, no ceremony. | v5 | 🛡 |

## Ticket ops

| skill | description | version | |
|---|---|---|---|
| [launifycorp/jira-sprint-summary](ticket-ops/jira-sprint-summary.md) | The outward-facing report after a sprint closes: a verdict against the sprint goal, committed versus delivered, blockers with a measured duration and a named dependency, hours grouped by work area with coverage stated first, and at most three improvements that each cite a number. Never per person. | v1 | 🛡 |
| [opsmith/jira-ticket-scorer](ticket-ops/jira-ticket-scorer.md) | Scores every Jira ticket for clarity, scope and effort before it hits the sprint. Flags vague acceptance criteria and suggests a rewrite. | v5 | ★ 🛡 |
| [opsmith/sprint-retro-facilitator](ticket-ops/sprint-retro-facilitator.md) | Prepares a retrospective from the sprint's data — what shipped, what slipped, where time went — and turns it into three concrete questions for the team. | v3 | 🛡 |

★ Emdly recommended · 🛡 Security checked (automated checks + clean AI safety scan + human review)
