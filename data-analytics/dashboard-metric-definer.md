---
name: dashboard-metric-definer
owner: querydeck
category: Data & analytics
description: Turns a vague metric request ("active users", "churn") into a precise definition with grain, window, filters and the SQL skeleton — before anyone builds the chart.
version: v2
license: MIT
updated: 2026-08-20
recommended: false
security_checked: true
url: https://emdly.com/skills/querydeck/dashboard-metric-definer
raw: https://emdly.com/raw/querydeck/dashboard-metric-definer.md
install: npx @emdly/cli add querydeck/dashboard-metric-definer
---

# Dashboard metric definer

Half of all dashboard arguments are two people using one word for two numbers. This skill writes the definition down before the chart exists.

## When to use
- When a metric is requested ("add churn to the exec dashboard").
- When two dashboards disagree and nobody knows why.

## Input
The requested metric in the requester's words, the tables available (names and key columns), and any existing definition in the company's glossary.

## Definition template
- **Name** — the one the glossary uses, or the plainest one.
- **Question it answers** — one sentence a non-analyst would say.
- **Grain** — per what: user, account, order, day.
- **Population** — who counts (paying? trial? internal accounts excluded?).
- **Event / condition** — what makes a unit count.
- **Window** — calendar month? trailing 28 days? As of when?
- **Edge rules** — reactivations, refunds, deleted accounts, time zones.
- **SQL skeleton** — the shape, with the tables named and placeholders for anything not decided.
- **Known alternatives** — the other definitions people might mean, with the number each would produce if the data lets you estimate it.

## Rules
- Ask, don't assume: when the request leaves a choice open (window, population), list the choice explicitly as a question with a recommended default. Never silently pick.
- If a glossary definition exists, use it and mark any deviation.
- Show the two most likely definitions side by side when they differ by more than 10% on a sample — that gap is the reason the meeting happened.
- Time zones and "as of" dates are never implicit.

## Output format
```
## Metric: Monthly active accounts
**Question:** How many paying accounts did something in the product last month?
**Grain:** account · **Population:** status = active, plan ≠ internal
**Event:** ≥ 1 event in `events` with type in (login, api_call) · **Window:** calendar month, UTC
**Edge rules:** an account that cancels mid-month still counts for that month

**Open choices**
1. Should `api_call` count as activity? Recommended: yes (integration-only customers exist).

**SQL skeleton**
SELECT date_trunc('month', e.created_at) AS month, COUNT(DISTINCT a.id) …

**Alternative:** "any account incl. trials" → ~2.3× the number. Not recommended for the exec view.
```

## License
MIT
