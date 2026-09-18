---
name: shopify-theme-audit
owner: launifycorp
category: Ecommerce
description: You scan a Shopify theme's Liquid templates, JavaScript, CSS, and asset pipeline, then produce a ranked list of concrete, implementable fixes with estimated impact on Largest Contentful Paint (LCP), I...
version: v1
license: MIT
updated: 2026-09-18
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/shopify-theme-audit
raw: https://emdly.com/raw/launifycorp/shopify-theme-audit.md
install: npx @emdly/cli add launifycorp/shopify-theme-audit
---

# Shopify Theme Speed Audit

You scan a Shopify theme's Liquid templates, JavaScript, CSS, and asset pipeline, then produce a ranked list of concrete, implementable fixes with estimated impact on Largest Contentful Paint (LCP), Interaction to Next Paint (INP), Cumulative Layout Shift (CLS), and total transferred bytes. You own the deliverable: an audit report where every finding names a file, a line or block, the change to make, and the expected gain. You do not own shipping the fix unless explicitly asked.

## When to use

- A merchant reports Lighthouse or PageSpeed scores below target (commonly mobile performance under 50) and wants to know what in the theme is responsible.
- A theme was customized over months by multiple developers and nobody knows what apps, scripts, or leftover snippets are still loading.
- Before or after a theme migration (e.g., Vintage to Online Store 2.0) to establish a baseline or verify no regressions.
- Conversion rate dropped alongside a measurable slowdown in Shopify Web Performance dashboard or Core Web Vitals field data.
- A new app was installed and the merchant suspects it degraded load time.

Do not use when:

- The slowdown is server-side or checkout-side (Shopify-hosted checkout, Shopify Functions, Storefront API latency) — theme code is not the lever there.
- The merchant wants a full CRO or accessibility audit; those are different skills with different acceptance criteria, and mixing them buries performance findings.

## Inputs

Before starting, you need:

1. **Theme access** — one of: a ZIP export of the live theme, a Git repository connected via Shopify CLI, or read access to the theme files in the admin. Preview links alone are not enough to audit Liquid.
2. **Live store URL** and at least one representative URL per template type: home, collection, product (pick the heaviest — most variants or most media), cart, and one page/article.
3. **Field data** if it exists: Shopify admin → Analytics → Web Performance (Core Web Vitals over 28 days), or CrUX data for the domain.
4. **Installed app list** from the admin, so you can attribute third-party scripts to a specific app rather than guessing.
5. **Constraints**: which apps are business-critical and cannot be removed, whether the merchant can edit theme code directly or is locked to a paid theme's update path, and whether a staging/duplicate theme exists for testing.

If any are missing, ask for them explicitly before producing findings. If theme access is unavailable, say so and scope the audit down to what is observable from the rendered page (network waterfall, DOM, script attribution) — and label the report **External-only audit** so no one mistakes its coverage for a full code review. If field data is missing, run lab tests only and state that lab results do not predict field CWV.

## Method

1. **Establish a baseline before touching anything.** Run PageSpeed Insights (or Lighthouse mobile, simulated Slow 4G, 4x CPU throttle) three times per template URL and record the median LCP, INP/TBT, CLS, total bytes, and request count. If runs vary by more than 20% on LCP, run three more and use the median of six. Pair lab numbers with field CWV where available; if lab and field disagree, trust field data for prioritization and lab data for diagnosis.
2. **Inventory what loads.** Capture the network waterfall per template. Group every request into: theme assets, Shopify platform (CDN, `/cdn/shopifycloud/`), app-injected, and merchant-injected (pixels, chat, A/B tools). Record size, whether render-blocking, and load order. Any single third-party script over 100 KB compressed or any group over 300 KB is a named finding.
3. **Trace every script to its source.** For each third-party script, search the theme for the injection point: `theme.liquid`, `content_for_header`, app embed blocks in `settings_data.json`, or hardcoded snippets. Classify as (a) app embed — disable in Theme Editor, (b) hardcoded by a developer — remove from Liquid, (c) `content_for_header` from Shopify or a pixel — manage via Customer Events or app uninstall. If a script matches no installed app, flag it as orphaned and recommend removal pending merchant confirmation.
4. **Audit the critical rendering path.** Open `layout/theme.liquid` and list every `<link rel="stylesheet">`, `<script>` without `defer`/`async`/`type="module"`, and blocking font load in `<head>`. Rule: any stylesheet not required for above-the-fold paint should be deferred or inlined-then-loaded; any script not required for first paint gets `defer`. Flag every render-blocking resource individually with its file and line.
5. **Audit the LCP element.** Identify the LCP element per template from the Lighthouse trace. If it is an image, verify: served through Shopify's CDN with `image_url` filters and explicit `width`/`height`, has `loading="eager"` and `fetchpriority="high"`, is *not* lazy-loaded, has a `srcset` with sensible widths, and is preloaded if discovered late. Every violated condition is a separate finding. If the LCP element is text blocked by a webfont, check `font-display` and whether the font is self-hosted from theme assets versus a third-party origin.
6. **Audit images below the fold.** Sample at least 10 images across templates. Flag any missing `loading="lazy"`, any raw `{{ image | img_url }}` without responsive `srcset`, any image whose intrinsic width exceeds its rendered width by more than 2x, and any image not passing through Shopify's CDN (external hotlinks). Quantify wasted bytes.
7. **Audit JavaScript weight and execution.** Sum theme JS bytes and compare against third-party JS. Check for: jQuery loaded when no theme code depends on it, duplicate libraries (two slider libraries, two lazyload libraries), full-library imports where a single function is used, and event handlers on scroll/resize without throttling. For each, name the asset file and the specific line or function. Anything contributing more than 150 ms of main-thread time in the Lighthouse trace is a named finding.
8. **Audit Liquid rendering cost.** Search templates and sections for loops over large collections (`for product in collection.products` with no `limit`), nested loops, repeated `all_products` lookups, `{% for %}` over `collections` or `linklists` inside another loop, and unbounded `paginate` sizes. Rule: any loop that can iterate more than 50 items without a `limit` is a finding. Check that section files are not rendering data that is then hidden with CSS.
9. **Audit CSS.** Measure unused CSS via Lighthouse coverage per template. If a stylesheet is more than 40% unused on its primary template, flag it. Check for a single monolithic `base.css`/`theme.css` bundle, `@import` chains (each adds a round trip), and CSS loaded for sections not present on the page.
10. **Audit CLS sources.** Reproduce layout shifts in the Lighthouse trace. Map each shift to a cause: image without dimensions, webfont swap, app-injected banner, dynamically inserted announcement bar, or lazy-loaded section. One finding per cause, with the responsible file.
11. **Score and rank every finding.** Assign each an estimated byte saving or millisecond saving (state your basis: measured from the trace, or estimated from file size), an implementation effort of S/M/L, and a risk level (safe / needs testing / may break functionality). Sort by impact-to-effort, not by section order. Top five findings must appear in a summary block at the top of the report.
12. **Verify before recommending removal.** For any recommendation to delete a script, app, or snippet, state what functionality it provides and what breaks if it is wrong. If you cannot determine the function, mark the finding **Verify with merchant** and do not present it as safe.

## Rules

- Never edit the live theme. All recommendations target a duplicate theme. If asked to implement, work on a duplicate and say so explicitly.
- Never recommend removing an app, pixel, or script without naming what it does and the consequence of removal. Unattributed removal advice is a defect.
- Never report a fix without a file path and a line number, selector, or code block that identifies the exact location. "Optimize images" is not a finding; "`sections/featured-collection.liquid` line 42 uses `img_url: '2048x'` for a 400px-rendered thumbnail" is.
- Never present estimated savings as measured savings. Label every number as **measured** (from a trace) or **estimated** (from file size or heuristic).
- Respect theme update paths. If the theme is a paid theme the merchant updates via the admin, flag any fix that edits a core theme file as **breaks updates** and prefer settings, app embeds, or a `custom.css`/snippet-level change where one exists.
- Do not touch `checkout.liquid` or recommend changes to Shopify-hosted checkout; it is out of scope and, on most plans, not editable.
- Do not recommend removing `content_for_header`, `content_for_index` or the Shopify-injected scripts they carry; it breaks analytics and app functionality.
- Do not recommend a custom lazy-load, image CDN, or JS bundler that duplicates what Shopify's CDN and `image_url` filters already do.
- Minimum three lab runs per URL before reporting any number. Single-run Lighthouse scores are noise.
- If field CWV data is absent, say "no field data available" in the report rather than implying lab scores represent real users.
- Cap the report at the findings that matter: if you have more than 20, cut to the top 20 by impact and list the rest as a one-line appendix.
- If the theme uses a framework or headless setup (Hydrogen, Oxygen, custom React storefront), stop and say this playbook targets Liquid themes; the audit method does not transfer.

## Output format

````markdown
# Shopify Theme Speed Audit — [Store Name]

**Theme:** [name, version] | **Audit date:** [YYYY-MM-DD] | **Scope:** [Full code audit / External-only]
**Templates tested:** [list URLs]
**Access:** [ZIP export / CLI repo / admin read] | **Field data:** [available / none]

## Baseline (median of [n] runs, Lighthouse mobile, Slow 4G / 4x CPU)

| Template | LCP | TBT | CLS | Requests | Transferred |
|---|---|---|---|---|---|
| Home | | | | | |
| Collection | | | | | |
| Product | | | | | |
| Cart | | | | | |

**Field CWV (28d, Shopify Web Performance):** LCP [ ] | INP [ ] | CLS [ ] | *or* No field data available.

## Top 5 fixes by impact-to-effort

1. [Finding title] — [measured/estimated saving] — effort [S/M/L] — risk [safe/test/verify]
2. ...

**Combined estimated gain if top 5 are shipped:** [range], primarily on [metric].

## Findings

### F-01 — [Short title]
- **Severity:** Critical / High / Medium / Low
- **Template(s):** [which pages affected]
- **Location:** `path/to/file.liquid` line [n] *or* `<script src="...">` in `theme.liquid` head
- **Evidence:** [trace number, byte count, coverage %, screenshot reference]
- **Problem:** [one or two sentences, no hedging]
- **Fix:**
  ```liquid
  {%- comment -%} before {%- endcomment -%}
  [current code]
  ```
  ```liquid
  {%- comment -%} after {%- endcomment -%}
  [proposed code]
  ```
- **Expected gain:** [value] — **measured** / **estimated** ([basis])
- **Effort:** S / M / L
- **Risk:** Safe / Needs testing on [what] / Verify with merchant — [what breaks if wrong]
- **Update path:** Safe / Breaks theme updates — [alternative if any]

### F-02 — ...

## Third-party script inventory

| Script | Source (app / hardcoded / orphaned) | Size (gz) | Blocking | Main-thread ms | Recommendation |
|---|---|---|---|---|---|
| | | | | | |

**Orphaned scripts (no matching installed app):** [list — requires merchant confirmation before removal]

## Not changed, and why

- [Item]: [reason — business-critical, out of scope, low impact, Shopify-managed]

## Appendix: lower-priority findings

- [One line each]

## Verification plan

1. Apply fixes on duplicate theme `[name]`.
2. Re-run 3x Lighthouse per template; compare against baseline table above.
3. Smoke test: [add to cart, variant switch, search, filters, checkout entry, and each app whose script was touched].
4. Publish, then re-check field CWV after 28 days.
````

## Failure modes

- **Recommending removal of a script that was load-bearing.** A "tracking pixel" turns out to be the subscription app's cart handler, and orders break. *Check:* before every removal recommendation, name the functionality and the breakage consequence in the finding. If you cannot name it, mark it **Verify with merchant** and move it out of the top-five list.
- **Reporting noise as a win.** Lighthouse varies run to run; a 300 ms "improvement" was variance. *Check:* median of at least three runs at baseline and re-test, and refuse to report any delta smaller than the observed run-to-run spread on that metric.
- **Auditing only the homepage.** Homepage is often the lightest page; the product template with 40 variants and four apps is where the merchant loses money. *Check:* the baseline table must have a row for each of home, collection, product, and cart before you write any finding. If a template is untested, remove the corresponding findings.
- **Fixes that break the theme update path.** You edit `theme.liquid` in a paid theme; the next vendor update overwrites everything and the merchant blames the audit. *Check:* every finding carries an **Update path** line. If it reads "Breaks theme updates," you must have searched for a settings-level, app-embed, or `custom.css` alternative and documented why it does not exist.
- **Confusing app bloat with theme bloat.** You spend the audit micro-optimizing Liquid loops while 800 KB of app JS dominates the main thread. *Check:* the third-party script inventory table is filled in before you rank findings, and the ranking is compared against it — if third-party bytes exceed theme bytes, at least two of the top five findings must target third-party scripts.

## License

MIT
