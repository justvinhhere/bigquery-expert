# SimpleSelectStar

**Severity:** High

## What It Detects
`SELECT *` on a simple single-table query that does not involve JOINs or GROUP BY.

**Not flagged:** `SELECT *` in queries with JOINs, GROUP BY, or subqueries -- these may legitimately need all columns.

## Why It Matters
BigQuery is a columnar storage engine. `SELECT *` reads every column in the table, dramatically increasing bytes scanned and cost. For wide tables (50+ columns), this can be orders of magnitude more expensive than selecting only the needed columns.

## Before
```sql
SELECT *
FROM `project.dataset.table1`
```

## After
```sql
SELECT id, name, email, created_at
FROM `project.dataset.table1`
```

## Edge Cases
- `SELECT * EXCEPT(col)` is acceptable when you need most columns.
- `SELECT *` inside a CTE or subquery that is later filtered is less impactful but still suboptimal.
- Exploratory queries with `LIMIT` are acceptable during development.
