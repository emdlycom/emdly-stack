---
name: dashboard-metric-definer
owner: querydeck
category: Data & analytics
description: Turns a vague metric request ("active users", "churn") into a precise definition with grain, window, filters and the SQL skeleton — before anyone builds the chart.
version: v3
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/querydeck/dashboard-metric-definer
raw: https://emdly.com/raw/querydeck/dashboard-metric-definer.md
install: npx @emdly/cli add querydeck/dashboard-metric-definer
---

# Dashboard metric definer

Half of all dashboard arguments are two people using one word for two numbers. This skill writes the definition down before the chart exists. The reader is the analyst who will build it and the executive who will quote it, so the definition has to be precise enough that two people working from it independently produce the same number, and honest enough to say when the tables on hand cannot produce the metric at all.

## When to use
- When a metric is requested in words — "add churn to the exec dashboard", "how many active users".
- When two dashboards disagree on one word and nobody knows why.
- Before a metric goes into a board deck, an investor update, or a contract.
- When a metric already on a dashboard has no written definition.
- Not to build the chart. This produces the definition and the SQL skeleton; someone else implements it.

## Input

Required:
- **The request in the requester's own words**, unedited. The ambiguity in the phrasing is the material.
- **The tables available** — names, key columns, types, nullability, and how they join. Enough to tell whether the metric is derivable, which is step 1 of the process.
- **The audience and the surface** — exec dashboard, team dashboard, investor update, contract or SLA. This decides how much of `## Stop and hand back` applies.

Wanted, and named when absent:
- **The company glossary** or any existing written definition of this metric.
- **Existing implementations** — the SQL behind any chart that already claims this metric.
- **A sample or row counts**, enough to estimate what each candidate definition returns.
- **The decision the metric supports.** "Which number would change what you do" resolves more open choices than any amount of analysis.

Without the glossary, the definition is still written, marked `no glossary supplied — this becomes the definition of record if accepted`. Without the tables, the skill cannot run: it would produce a template, not a definition. Report `tables not supplied` and stop.

## Definition template

Work through it in this order. Step 1 can end the job.

1. **Derivability check.** Before defining anything, name the columns each part of the metric needs and find them in the supplied tables. If any part has no source, stop here and produce the `NOT DERIVABLE` output below.
2. **Name** — the one the glossary uses, or the plainest one. If it collides with an existing metric of a different definition, rename this one rather than overloading the word.
3. **Question it answers** — one sentence a non-analyst would say out loud.
4. **Grain** — per what: user, account, order, day, account-month.
5. **Population** — who counts. Paying? Trial? Internal and test accounts excluded? Name the column and value for each.
6. **Event / condition** — what makes a unit count, as a predicate on named columns.
7. **Window** — calendar month, trailing 28 days, quarter to date. And "as of" what: the event timestamp, or the state at period end.
8. **Time zone** — explicit, always. Name it.
9. **Edge rules** — reactivations, refunds, upgrades mid-period, deleted accounts, backfilled rows, late-arriving events, accounts that change plan.
10. **SQL skeleton** — the shape, with real table and column names and a placeholder for every choice not yet made.
11. **Known alternatives** — the other definitions people might mean, each with the number it produces on the sample when the data allows.
12. **Open choices** — every choice the request left open, as a question with a recommended default and the reason.

## Rules

- Ask, do not assume. Any choice the request leaves open (window, population, "as of") appears in Open choices with a recommended default. Never silently pick one.
- If a glossary definition exists, use it. Mark every deviation and say why.
- **Report every candidate definition's number. There is no divergence threshold and you must not assert one.** A 3% gap on a churn number quoted to a board matters; a 40% gap on an exploratory chart may not. Compute each candidate on the sample, print both numbers and the percentage difference, and let the owner decide whether the gap matters. Do not filter alternatives by size of gap.
- Where the sample cannot support an estimate, write `not estimable on this sample` next to the alternative rather than dropping it.
- Time zone and "as of" are never implicit. A definition without both is incomplete and is not published.
- Every population and event rule cites a column that exists in the supplied tables. No rule may rest on a column you have not seen.
- Estimate on at least 3 complete periods of history (house rule — one period cannot show whether a gap between definitions is structural or seasonal). With fewer, say so and mark the estimates `single period, seasonality not separable`.

Thresholds above are defaults; report the thresholds you used.

## Output format

```
## Metric: Monthly active accounts
Glossary: no entry for "active". No glossary supplied — this becomes the definition of
record if accepted. Estimates run on 6 complete months (Mar–Aug 2026), UTC.

**Question:** How many paying accounts did something in the product last month?
**Grain:** account-month · one row per account per calendar month.
**Population:** `accounts.status = 'active'` AND `accounts.plan NOT IN ('internal','test')`
**Event:** ≥ 1 row in `events` with `type IN ('login','api_call')`
**Window:** calendar month · **Time zone:** UTC · **As of:** the event timestamp
**Edge rules:**
- An account that cancels mid-month still counts for that month (it was active when the
  event happened).
- Reactivation inside the same month counts once, not twice — `COUNT(DISTINCT)`.
- Events with `created_at` backfilled after month end: (not stated) — no backfill flag
  exists in `events`. This edge rule is unresolved and is Open choice 3.

**Numbers, all candidates** — August 2026:
| definition | accounts | vs recommended |
| --- | --- | --- |
| calendar month, paying only (recommended) | 4 812 | — |
| trailing 28 days, paying only | 4 655 | −157 (−3.3%) |
| calendar month, incl. trials | 11 240 | +6 428 (+133.6%, 2.34×) |
| login only, no api_call | not estimable on this sample — `events.type` was not
  included in the extract |

Both of the first two are reported because the gap is reported, not judged. 3.3% on a
number the board reads month over month is a real difference; decide it deliberately.

**SQL skeleton**
SELECT date_trunc('month', e.created_at AT TIME ZONE 'UTC') AS month,
       COUNT(DISTINCT a.id) AS active_accounts
FROM events e
JOIN accounts a ON a.id = e.account_id
WHERE e.type IN (<< 'login','api_call' — Open choice 1 >>)
  AND a.status = 'active'
  AND a.plan NOT IN ('internal','test')
GROUP BY 1;

**Open choices**
1. Does `api_call` count as activity? Recommended: yes — integration-only customers
   exist and are real usage. Costs +unknown accounts; `type` was not in the extract.
2. Calendar month or trailing 28 days? Recommended: calendar month, because the board
   deck is monthly. Difference: 157 accounts (−3.3%).
3. Late-arriving events. There is no backfill flag, so a month's number changes if it is
   re-run later. Recommended: freeze the number 3 days after month end and store it.
   This one needs a decision before the chart is built, not after.

**Handed to:** whoever owns the exec dashboard. Not published until choices 1–3 are
answered — see Stop and hand back.
```

When the tables cannot produce the metric, that is the output:

```
## Metric: Net revenue retention — NOT DERIVABLE FROM THE TABLES SUPPLIED

**Requested:** "add NRR to the exec dashboard"
**Needs:** revenue per account per period, expansion and contraction within a cohort,
and churned revenue, all at account-month grain.

| component | source found |
| --- | --- |
| account list, plan, status | `accounts` — present |
| period boundaries | derivable from `subscriptions.started_at` — present |
| revenue per account per month | NOT FOUND. `subscriptions` holds `plan_name` and
  `started_at`/`ended_at`; there is no amount, no price, and no invoice table. |
| discounts, credits, refunds | NOT FOUND. No billing table supplied. |

**Not doing this instead:** a plan-count proxy ("accounts on a higher plan than last
year") is not NRR and will be quoted as if it were. It is off by every discount,
mid-term change and refund in the book.

**To unblock, one of:**
- Supply the billing tables (invoices or charges, with amount, currency, period, and
  credit/refund rows), or
- Agree a different metric that the current tables do support — logo retention at
  account grain is derivable today from `accounts` and `subscriptions`.

**Handed to:** the data owner for billing, plus whoever asked for NRR. Nothing was
defined and nothing should be charted from this request yet.
```

## Edge cases

- **The metric is not derivable from the tables given.** Use the output above. Name every missing component and the column it would need, name the proxy you are refusing and what it would be wrong by, and give the two unblock paths. Never define the nearest derivable thing under the requested name.
- **Partly derivable.** Define the part that is sourced, mark the rest `not derivable`, and do not average or interpolate across the gap. Say which decisions the partial metric can and cannot support.
- **No tables supplied.** Report `tables not supplied` and stop. Anything produced would be a template with invented column names, which reads like a definition and is not one.
- **No glossary.** Proceed, marked `no glossary supplied — this becomes the definition of record if accepted`.
- **Glossary and existing implementation disagree.** Report both, with both numbers if estimable. Do not pick. This is exactly the argument the skill exists to surface, and it belongs to the metric's owner.
- **The request is one ambiguous word** — "engagement", "health", "activity". Do not define it. Return the two or three metrics it could mean, each with its question sentence, and ask which decision the number supports.
- **Two metrics in one request** ("churn and retention"). Split into two definitions. They are not complements once refunds and mid-period plan changes exist.
- **Under 3 complete periods of history.** Estimates still run, marked `single period, seasonality not separable`. Do not report a trend from them.
- **Empty or near-empty sample** — no rows in the window, or too few to distinguish candidates. Write `too small to judge` in the numbers table rather than a figure, and keep every alternative listed.
- **Tables too large to sample directly.** Estimate on one complete period, state the period, and mark every figure as single-period. Do not extrapolate a monthly figure from a partial month; the shape of a month is not linear.
- **A column exists but is unpopulated** (all NULL, or only filled after some date). Treat it as absent for any period before it was populated, and say from which date the definition is computable.
- **The requester supplies the SQL and asks for the definition.** Reverse the template from the query, then check it against the request. Where the SQL and the request disagree, that gap is the finding.

## Stop and hand back

A metric definition is quoted long after this file is forgotten, and it reaches boards, investors and contracts without another review. Stop, publish nothing, and name who decides.

- **Any revenue metric** — ARR, MRR, NRR, churn, LTV, CAC, margin. Draft it, then hand to finance for sign-off before it appears anywhere. Finance owns the revenue recognition rules, and they are not derivable from the product tables.
- **A metric going into a board deck, an investor update, a fundraise, or a public statement.** The definition ships with it, or it does not ship. Hand to whoever signs the document.
- **A metric in a contract or an SLA** — uptime, response time, usage entitlements, anything that triggers a credit or a penalty. The contract's wording governs, not the convenient definition. Hand to whoever owns the contract, with the gap between the two spelled out.
- **A metric that decides pay** — commission, bonus, quota attainment — or that ranks people. Hand to whoever owns compensation. Never resolve an open choice yourself on a metric that pays someone.
- **The metric is not derivable** and someone has asked for a proxy anyway. Produce the NOT DERIVABLE output and stop. The decision to publish a proxy under the real metric's name is not yours; name it as a proxy, and hand it back.
- **The new definition changes an already-published number.** Do not quietly redefine. Report the old number, the new number, and the periods affected, and hand the restatement decision to the metric's owner.
- **Open choices unanswered.** The definition is not final while any Open choice is open. Hand it back with the choices listed; do not let a recommended default become the answer by silence.

## License
MIT
