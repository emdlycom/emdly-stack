---
name: sql-query-explainer
owner: querydeck
category: Data & analytics
description: Explains what a query actually does — joins, filters, gotchas — in plain language, then flags the index it wishes existed.
version: v3
license: MIT
updated: 2026-08-06
recommended: true
security_checked: true
url: https://emdly.com/skills/querydeck/sql-query-explainer
raw: https://emdly.com/raw/querydeck/sql-query-explainer.md
install: npx @emdly/cli add querydeck/sql-query-explainer
---

# SQL query explainer

Reads a query the way a reviewer should: what rows come out, which rows silently disappear, and where the time goes.

## When to use
- On any query in a PR, a dashboard, or a ticket ("why is this slow", "why is this number wrong").
- With `EXPLAIN (ANALYZE)` output when available — the explanation gets sharper.

## Input
The query, the dialect (PostgreSQL, MySQL, SQLite, BigQuery…), table definitions if available, and optionally the plan.

## Explanation, in this order
1. **One sentence:** what the result set is, in business terms ("one row per customer with their last paid invoice").
2. **Grain.** What one output row represents. If a join can multiply rows (one-to-many without aggregation), say so — this is the most common wrong-number bug.
3. **Filters that drop rows silently:** `INNER JOIN` on optional relations, `WHERE col = x` on a nullable column (NULLs vanish), `NOT IN` with NULLs (returns nothing), `DISTINCT` hiding a join fan-out.
4. **NULL semantics** wherever they change the answer.
5. **Where the time goes.** From the plan if given; otherwise from the shape: sequential scans on filtered columns, functions on indexed columns (`WHERE date(created_at) = …`), `ORDER BY` without a supporting index, `OR` across columns.
6. **The index it wishes existed** — as a concrete statement with the predicate it serves. Say when an index would *not* help (low selectivity, tiny table).

## Rules
- Never rewrite the query unless asked; explain it. If a rewrite is the fix, show the smallest one and state that the result set is identical (or how it differs).
- Dialect matters: say when a claim is dialect-specific.
- If tables are unknown, say which assumption the explanation rests on.

## Output format
```
**Result:** one row per customer who has at least one invoice, with the total of *paid* invoices.

**Grain:** customer. Safe — `SUM` aggregates the invoices join.

**Silent drops:** `INNER JOIN invoices` removes customers with no invoices; `WHERE status = 'paid'` moves into the join filter — customers with only unpaid invoices also vanish. If you want them at 0, use `LEFT JOIN … AND status = 'paid'`.

**Time:** `WHERE date(created_at) = current_date` prevents the index on `created_at`. Use a range: `created_at >= current_date AND created_at < current_date + 1`.

**Index:** `CREATE INDEX invoices_customer_status ON invoices (customer_id, status) WHERE status = 'paid';` — serves the join + filter. (PostgreSQL partial index.)
```

## License
MIT
