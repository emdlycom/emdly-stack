---
name: sklik-campaign-audit
owner: launifycorp
category: Marketing
description: You audit a Sklik (Seznam) advertising account and produce a prioritised findings list covering wasted spend, broken or rejected ads, and bidding misconfiguration. You own the deliverable: a written a...
version: v1
license: MIT
updated: 2026-09-15
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/sklik-campaign-audit
raw: https://emdly.com/raw/launifycorp/sklik-campaign-audit.md
install: npx @emdly/cli add launifycorp/sklik-campaign-audit
---

# Sklik Campaign Audit

You audit a Sklik (Seznam) advertising account and produce a prioritised findings list covering wasted spend, broken or rejected ads, and bidding misconfiguration. You own the deliverable: a written audit with quantified waste in CZK, a specific fix per finding, and an owner-ready action list. You do not implement changes unless explicitly asked in a separate task.

## When to use

- Monthly or quarterly account review where spend exceeds roughly 20 000 CZK/month and no audit has been run in 60+ days.
- Cost per conversion or ROAS has degraded by more than 20% versus the prior comparable period.
- An account is being taken over from another agency, freelancer, or internal owner.
- Spend is rising while conversions are flat or falling, and the cause is unknown.
- Before a budget increase, to confirm the account can absorb more spend without amplifying existing waste.

Do not use when:

- The account has run for fewer than 14 days or has under 300 total clicks in the period — data is too thin to separate noise from waste.
- The request is campaign build, keyword research, or ad copywriting; this skill only diagnoses an existing account.

## Inputs

Before starting, collect:

1. **Account access or exports** — Sklik interface access (read is enough) or CSV exports for: campaigns, ad groups, keywords, ads, search terms ("hledané dotazy"), placements/sites (content network), and conversions.
2. **Date range** — a primary window of at least 30 days, plus the equal preceding window for comparison.
3. **Conversion setup facts** — which conversion actions exist, which are counted in the "conversions" column, conversion value tracking on/off, attribution window.
4. **Business targets** — target CPA or ROAS in CZK, monthly budget cap, and which campaigns are brand vs. non-brand.
5. **Landing page list** — the destination URLs in use, or permission to crawl them.

If anything is missing, ask for it explicitly and state what you cannot conclude without it. Minimum viable set: campaign, keyword, search-term, and ad exports for 30 days. If targets are missing, ask once; if unavailable, derive an internal benchmark as the account median CPA over the period and label every target-based finding as "benchmark-derived, unconfirmed". Never invent a target.

## Method

1. **Fix the scope and reconcile totals.** Record date range, currency (CZK), and total spend, clicks, impressions, conversions, conversion value. Sum campaign-level spend and check it matches account-level spend within 1%. If it does not, the export is filtered or partial — request a clean export before continuing.
2. **Map account structure.** List campaigns with network (search / content / product ads / DSA / retargeting / video), status, daily budget, bidding strategy, and spend share. Flag any campaign mixing search and content in one campaign as a structural finding. Flag any campaign consuming more than 40% of spend for deeper inspection.
3. **Check conversion tracking integrity first.** If any campaign with more than 5 000 CZK spend records zero conversions and is not a declared awareness campaign, treat tracking as suspect. Verify the conversion code fires on the relevant thank-you pages if you have access. If tracking is broken, mark all performance findings "provisional" — do not recommend pausing on the basis of untracked data.
4. **Find zero-conversion spend.** For each keyword, ad group, placement, and campaign, compute spend and conversions. Flag any entity with spend ≥ 3× target CPA (or 3× benchmark CPA) and zero conversions. Sum this as "confirmed zero-return spend". Decision rule: recommend pause for entities ≥ 5× target CPA with zero conversions; recommend bid reduction of 30–50% for entities between 3× and 5×.
5. **Find over-target spend.** Flag entities with conversions but CPA above 1.5× target, or ROAS below 60% of target. Decision rule: if volume is ≥ 10 conversions, recommend bid or target adjustment proportional to the gap; if under 10 conversions, mark as "monitor — insufficient volume" rather than act.
6. **Audit search terms.** Pull the search terms report, sort by spend descending, and review the terms covering the top 70% of search-term spend plus every term with spend ≥ 2× target CPA and zero conversions. Classify each as relevant / irrelevant / ambiguous. Every irrelevant term becomes a negative keyword recommendation, placed at the tightest level that does not block relevant traffic. Quantify irrelevant-term spend as a CZK total.
7. **Audit content-network placements.** For content and retargeting campaigns, sort sites by spend. Flag placements with spend ≥ 2× target CPA and zero conversions, and any placement with CTR above 3× campaign average combined with zero conversions (accidental-click pattern). Recommend exclusion.
8. **Audit keyword match types and duplicates.** List broad-match keywords with no phrase or exact counterpart. Flag identical keywords appearing in more than one ad group in the same campaign group; identify which one won impressions and recommend removing the loser. Flag keywords with quality score ("skóre kvality") below 4 and spend above 1 000 CZK.
9. **Audit ads for breakage.** For every enabled ad group, check: number of enabled ads (flag ad groups with fewer than 2, and text-ad groups with 0), rejected or limited ads, ads with placeholder text, ads whose destination URL 404s, redirects off-domain, or lands on a non-HTTPS page. Check every landing URL with an HTTP request where possible; record status codes. Flag mismatch between ad promise (price, discount, product) and landing page content when observable.
10. **Audit extensions and ad assets.** Check for missing sitelinks, callouts, phone, and location extensions at campaign level. Flag campaigns with zero extensions as a finding with estimated CTR upside stated as a range, not a promise.
11. **Audit bidding configuration.** For each campaign record bidding strategy, target CPA/ROAS value, daily budget, and whether the budget is limited ("omezeno rozpočtem"). Apply these rules: automated CPA bidding with fewer than 15 conversions in 30 days is under-fed — recommend manual or lower-friction strategy; target CPA set more than 30% below actual 90-day CPA is throttling volume — flag; a budget-limited campaign hitting target CPA is a budget-raise candidate; a budget-limited campaign missing target is a bid-reduction candidate, not a budget-raise. Flag manual CPC campaigns where max CPC exceeds target CPA × conversion rate — mathematically loss-making.
12. **Check schedule, geo, and device.** Compare CPA by day-part, region, and device where data allows. Flag any segment with spend ≥ 5 000 CZK and CPA above 2× account average as a bid-modifier candidate. Flag campaigns targeting outside the stated business geography.
13. **Quantify total recoverable spend.** Sum zero-return spend, irrelevant search-term spend, and wasted placement spend, de-duplicated so no CZK is counted twice. Express as an absolute CZK figure and a percentage of total spend.
14. **Prioritise.** Rank every finding by recoverable CZK descending, then by implementation effort ascending. Assign each a severity: Critical (broken tracking, 404 ads, loss-making bids), High (waste ≥ 5% of spend), Medium (waste 1–5%), Low (structural or hygiene).
15. **Write the deliverable** using the output template. Every finding must name the exact campaign / ad group / keyword / URL affected, state the CZK impact, and give one specific action.

## Rules

- Never change bids, budgets, statuses, or ads during an audit. Diagnosis only.
- Never report a finding without naming the specific entity and its CZK spend in the period.
- Never claim a conversion improvement percentage as a certainty; use ranges and label them as estimates.
- Do not recommend pausing anything with fewer than 100 clicks or less than 3× target CPA in spend — insufficient evidence.
- Do not treat brand campaigns and non-brand campaigns under the same CPA benchmark; separate them and say so.
- If conversion tracking is broken or unverifiable, say so in the first line of the summary and mark all performance findings provisional.
- If data for a required check is missing, list the check under "Not assessed" with the reason and the exact export needed. Never silently skip a section.
- Respect the date range given; do not mix windows within a single comparison.
- All figures in CZK, excluding VAT unless the export states otherwise; state which.
- Cap the action list at 15 items. If more exist, keep the highest-recoverable 15 and note the remainder count.

## Output format

```
# Sklik Account Audit — [Account name]
Period: [YYYY-MM-DD to YYYY-MM-DD] | Comparison: [YYYY-MM-DD to YYYY-MM-DD]
Prepared: [YYYY-MM-DD] | Currency: CZK ([incl./excl.] VAT)

## Summary
Tracking status: [Verified / Suspect / Broken — one sentence]
Total spend: [X] CZK | Conversions: [X] | CPA: [X] CZK | ROAS: [X or n/a]
Identified recoverable spend: [X] CZK ([X]% of total)
Top three issues: [1], [2], [3]

## Account snapshot
| Campaign | Network | Status | Bidding | Budget/day | Spend | Conv | CPA | Note |
|---|---|---|---|---|---|---|---|---|
| | | | | | | | | |

## Findings

### Critical
**C1. [Title]**
- Where: [campaign / ad group / keyword / URL]
- Evidence: [metrics with numbers]
- Impact: [X] CZK in period
- Action: [one specific instruction]

### High
**H1. [Title]**
- Where:
- Evidence:
- Impact: [X] CZK
- Action:

### Medium
**M1. [Title]**
- Where:
- Evidence:
- Impact: [X] CZK
- Action:

### Low
**L1. [Title]**
- Where:
- Evidence:
- Action:

## Wasted spend breakdown
| Source | Entities | Spend | Conversions | Recoverable CZK |
|---|---|---|---|---|
| Zero-conversion keywords | | | 0 | |
| Irrelevant search terms | | | | |
| Zero-conversion placements | | | 0 | |
| Over-target CPA entities | | | | |
| **Total (de-duplicated)** | | | | |

## Broken ads and landing pages
| Campaign | Ad group | Issue | URL | HTTP status | Action |
|---|---|---|---|---|---|

## Bidding issues
| Campaign | Strategy | Target | Actual CPA | 30d conversions | Diagnosis | Action |
|---|---|---|---|---|---|---|

## Negative keyword recommendations
| Term | Spend | Clicks | Conv | Add at level | Match type |
|---|---|---|---|---|---|

## Action list (ranked)
1. [Action] — [entity] — [CZK impact] — [effort: S/M/L] — [severity]
2.
...
[Remaining findings not listed: X]

## Not assessed
- [Check] — reason: [missing data] — needed: [exact export or access]

## Assumptions
- [Any benchmark-derived target, attribution assumption, or unverified input]
```

## Failure modes

- **Reporting waste that is actually untracked conversion value.** Phone orders, offline sales, or a broken conversion tag make profitable keywords look dead. Check: before flagging any zero-conversion entity, confirm step 3 passed and confirm no conversion action exists that is recorded but excluded from the "conversions" column. If tracking is suspect, every waste figure carries the "provisional" label.
- **Double-counting recoverable spend.** The same CZK gets counted as a zero-conversion keyword, an irrelevant search term, and an over-target ad group, inflating the total two or three times. Check: reconcile the de-duplicated total against total account spend — recoverable spend above 40% of total spend almost always means double counting; re-derive from unique keyword/placement IDs.
- **Recommending pauses on thin data.** A keyword with 12 clicks and zero conversions is not evidence of waste. Check: every pause recommendation must show clicks ≥ 100 or spend ≥ 3× target CPA; scan the action list and downgrade any item failing this to "monitor".
- **Missing broken ads because URLs were never requested.** Reading ad copy in the interface does not reveal 404s, redirects, or expired promo pages. Check: the broken-ads table must contain an HTTP status code for every unique destination URL in enabled ad groups; a blank status column means the check was not run.

## License

MIT
