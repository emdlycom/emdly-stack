---
name: sql-query-explainer
owner: querydeck
category: Data & analytics
description: Explains what a query actually does — joins, filters, gotchas — in plain language, then flags the index it wishes existed.
version: v4
license: MIT
updated: 2026-08-30
recommended: true
security_checked: true
url: https://emdly.com/skills/querydeck/sql-query-explainer
raw: https://emdly.com/raw/querydeck/sql-query-explainer.md
install: npx @emdly/cli add querydeck/sql-query-explainer
---

# SQL query explainer

Reads a query the way a reviewer should: what rows come out, which rows silently disappear, and where the time goes. The reader is whoever has to trust the number the query produces, or make it faster, so the explanation names the grain, names every place rows vanish without an error, and says out loud which assumptions it rests on when the table definitions are not in front of it.

## When to use
- On any query in a PR, a dashboard, or a ticket — "why is this slow", "why is this number wrong".
- With `EXPLAIN (ANALYZE, BUFFERS)` output when it exists; the time section gets sharper.
- Before a query becomes the definition of a reported metric.
- When two queries that should agree do not, and you need the grain of each.

## Input

Required:
- **The query**, complete. Including CTEs and any view it selects from, if you have them.
- **The dialect and version** — PostgreSQL 16, MySQL 8.0, SQLite, BigQuery, Snowflake. Behaviour below differs across all of them.

Strongly wanted, and named in the output when absent:
- **Table definitions** — columns, types, nullability, primary and foreign keys, existing indexes. Nullability and the uniqueness of join keys are what steps 2–4 are made of.
- **The plan** — `EXPLAIN ANALYZE` output, with row estimates and actual rows.
- **Row counts** per table, and roughly how selective the main filters are.

Optional: the query's purpose in one sentence from whoever wrote it, and the number it is currently producing versus the number expected.

Without table definitions the skill still runs, on stated assumptions — see step 2 of the rules and the second example. Without the dialect, do not guess: ask, or explain only what is true in all four and say which claims were withheld.

## Explanation, in this order

1. **One sentence:** what the result set is, in business terms ("one row per customer with their last paid invoice").
2. **Grain.** What one output row represents. If a join can multiply rows — one-to-many with no aggregation, or a join key that is not unique on the far side — say so. This is the most common wrong-number bug and it never raises an error.
3. **Filters that drop rows silently:** `INNER JOIN` on an optional relation; `WHERE col = x` on a nullable column, where NULLs vanish; `NOT IN (…)` against a set containing NULL, which returns nothing at all; `DISTINCT` hiding a fan-out rather than fixing it; a `WHERE` predicate on the outer table of a `LEFT JOIN`, which turns it back into an inner join.
4. **NULL semantics** wherever they change the answer: comparison, aggregation (`COUNT(col)` skips NULL, `COUNT(*)` does not), `GROUP BY` grouping NULLs together, sort position, and unique constraints.
5. **Where the time goes.** From the plan if given — the node whose actual time dominates, and where estimate and actual diverge. Otherwise from the shape: sequential scans on filtered columns, a function wrapped around an indexed column (`WHERE date(created_at) = …`), `ORDER BY` with no supporting index, `OR` across columns, a correlated subquery in the select list.
6. **The index it wishes existed** — as a concrete statement, with the predicate it serves. Say when an index would *not* help: low selectivity, a table small enough to scan, or a write-heavy table where the index costs more than it returns.

## Rules

- Never rewrite the query unless asked; explain it. If a rewrite is the fix, show the smallest one and state whether the result set is identical, or exactly how it differs.
- Dialect matters. Say when a claim is dialect-specific, and name the dialect it holds in.
- **If tables are unknown, say which assumption the explanation rests on.** Put it in an `Assumptions` block at the top, one line per assumption, each naming the column and what was assumed of it (nullable, unique, indexed), and each marked with what changes in the explanation if it is wrong.
- Separate what the plan shows from what the shape suggests. Label the second `[from shape, no plan]`.
- Do not estimate a runtime or a speed-up factor. You cannot. Say which node dominates and why.
- No claim about data distribution without row counts. "Low selectivity" needs a number or it is a guess; if you have none, say `selectivity unknown`.

Thresholds above are defaults; report the thresholds you used.

## Output format

An `Assumptions` block when anything was assumed, then the six sections above, in order.

```
**Assumptions:** none — DDL supplied for `customers` and `invoices`.

**Result:** one row per customer who has at least one invoice, with the total of *paid* invoices.

**Grain:** customer. Safe — `SUM` aggregates the invoices join, and `customers.id` is the primary key.

**Silent drops:** `INNER JOIN invoices` removes customers with no invoices; `WHERE status = 'paid'` moves into the join filter — customers with only unpaid invoices also vanish. If you want them at 0, use `LEFT JOIN … AND status = 'paid'`.

**NULLs:** `invoices.amount` is nullable. `SUM` skips NULL, so a customer whose invoices are all NULL-amount returns NULL, not 0. Wrap in `COALESCE(SUM(amount), 0)` if the dashboard expects a number.

**Time:** `WHERE date(created_at) = current_date` prevents the index on `created_at`. Use a range: `created_at >= current_date AND created_at < current_date + 1`.

**Index:** `CREATE INDEX invoices_customer_status ON invoices (customer_id, status) WHERE status = 'paid';` — serves the join + filter. (PostgreSQL partial index.)
```

When the tables are unknown the assumptions carry the explanation, and every one of them is stated:

```
**Assumptions** — no DDL supplied. This explanation rests on four, and two of them
change the answer if wrong:
1. `orders.customer_id` is nullable. If it is NOT NULL, the silent drop below disappears.
2. `refunds.order_id` is NOT unique — one order can have several refunds. If it is
   unique, the grain is safe and the fan-out below does not exist. *This is the one to
   check first: it is the difference between a right and a wrong revenue number.*
3. `orders.created_at` is a timestamp, not a date. If it is a date, the `>= / <` range
   is still correct and nothing changes.
4. No index on `refunds.order_id`. If one exists, the index recommendation is already met.

**Result:** one row per order placed this month, with its refunded amount.

**Grain:** NOT order — this is the bug. `JOIN refunds ON refunds.order_id = o.id` fans
out to one row per refund under assumption 2, so `SUM(o.total)` counts an order's total
once per refund. Two refunds on one order double-count it. `DISTINCT` would hide the
duplicate rows and leave the sum wrong.

**Silent drops:** `INNER JOIN refunds` keeps only refunded orders — under assumption 1
the un-refunded ones are gone, so this is not "orders this month". `NOT IN (SELECT
customer_id FROM blocklist)` returns zero rows if any `blocklist.customer_id` is NULL
(all four dialects). Use `NOT EXISTS`.

**NULLs:** `COUNT(r.id)` skips NULL, so under a `LEFT JOIN` rewrite it reports 0 for an
un-refunded order, while `COUNT(*)` would report 1. `COUNT(r.id)` is the one you want.

**Time:** [from plan] the dominant node is a Seq Scan on `refunds` (actual 1 190 ms of
1 340 ms total), estimated 500 rows, actual 812 000 — the estimate is off by ~1 600×,
so the planner chose a nested loop it should not have. `WHERE date(o.created_at) =
current_date` also blocks any index on `created_at`. Selectivity of `status = 'open'`
is unknown; no row counts supplied.

**Index:** `CREATE INDEX refunds_order_id ON refunds (order_id);` — serves the join, and
should turn the seq scan into an index scan. Conditional on assumption 4. If `refunds`
is under a few thousand rows the scan is already cheap and this index is not worth the
write cost; row counts were not supplied, so this is unresolved.
```

## Edge cases

- **Invalid SQL** — it does not parse, or references a keyword the dialect does not have. Do not explain it and do not guess the intent. Report `does not parse in <dialect>`, quote the smallest failing fragment with its line number, name the likely cause (unbalanced parenthesis, a trailing comma before `FROM`, `LIMIT` in T-SQL), and stop. An explanation of a query that cannot run is worse than nothing: it gets quoted as though the query ran.
- **Valid in another dialect** — `LIMIT` in Oracle, `TOP` in PostgreSQL, backtick quoting in PostgreSQL. Say which dialect it parses in and ask before continuing; do not silently explain it as though the dialect were the one it fits.
- **A very long query** — over 200 lines, or more than 10 joined relations (house rule; the limits are about what a reader can hold, not about the engine). Do not explain it clause by clause. Decompose first: list the CTEs and subqueries in dependency order, one line each for what each produces and at what grain, and give the whole thing a grain map. Then run the six steps on the top-level select and on any CTE whose grain changes. State that this is what you did and which CTEs you did not open.
- **A query that will not decompose** — hundreds of lines with no CTEs and deep inline subqueries. Report the grain map and the silent drops only, and say the time analysis needs a plan. Do not pretend to whole-query coverage.
- **Empty input, or a fragment** (a `WHERE` clause, a snippet with no `FROM`). Report `not a complete statement` and say what is missing. Do not complete it.
- **No plan supplied.** Section 5 still runs, entirely from the shape. Every claim in it is tagged `[from shape, no plan]`, and no claim about actual cost is made.
- **No table definitions.** Runs on stated assumptions, as in the second example. If a needed assumption cannot be made safely — a join key whose uniqueness decides the grain — do not pick a side: state both grains and say the answer depends on that one fact.
- **No dialect.** Explain only the behaviour common to the major dialects and list what was withheld: NULL handling in `NOT IN` is common, index syntax and partial-index support are not.
- **A view or function you cannot see.** Explain to that boundary. Mark its grain `unknown — definition of <name> not supplied`, and do not assume it returns one row per anything.
- **Generated SQL** (ORM, dbt, BI tool). Explain what it does, then say so: the fix usually belongs in the model, not in the SQL, and rewriting the output will be overwritten on the next build.
- **`SELECT *` with no DDL.** The output columns cannot be enumerated. Say `output columns unknown` rather than listing the ones the joins imply.

## Stop and hand back

The explanation is safe. What comes out of section 6 is not: an index statement is a production DDL change, and a grain finding may mean a published number is wrong.

- **Do not run anything.** This skill reads SQL. It never executes the query, never runs `EXPLAIN`, and never creates an index. If a plan is needed, ask for it.
- **Index recommendations go to whoever owns the migration.** Name them in the output. An index on a large or write-heavy table is a schema change with a lock and a cost, and it is decided by the database owner, not here.
- **A grain or silent-drop finding on a query behind a published number** — a board metric, an invoice, a payout, a regulatory report. Stop at the finding. Hand to the metric's owner with the two numbers (as-is, and as-corrected) and let them decide about restatement. Do not quietly supply a corrected query for someone to swap in.
- **The query writes** — `UPDATE`, `DELETE`, `INSERT`, `MERGE`, `TRUNCATE`, or DDL. Explain which rows it will touch and where the filter fails to catch some of them, then stop. Say plainly that it was not run and must not be run from this explanation. Whoever owns the table approves it.
- **The query exposes personal or regulated data** — it selects columns that look like identifiers, payment details, or health data across accounts. Do not reproduce sample values in the explanation, and flag the access-scope question to whoever owns the data.

## License
MIT
