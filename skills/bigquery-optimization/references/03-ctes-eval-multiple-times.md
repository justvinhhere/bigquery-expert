# CTEsEvalMultipleTimes

**Severity:** High

## What It Detects
A CTE (Common Table Expression, defined with `WITH`) whose alias is referenced more than once in the query. In BigQuery, CTEs are inlined at each reference point -- the entire CTE query is re-executed for every reference.

## Why It Matters
If a CTE performs expensive computation (aggregations, joins, large scans) and is referenced 2+ times, that computation runs 2+ times. This doubles (or more) the slot consumption and bytes processed.

## Before
```sql
WITH a AS (
  SELECT col1, col2
  FROM `project.dataset.table1`
  WHERE col1 = 'abc'
),
b AS (
  SELECT col2 FROM a
),
c AS (
  SELECT col1 FROM a
)
SELECT b.col2, c.col1
FROM b, c
```
CTE `a` is referenced 2 times -- it will be computed twice.

## After
```sql
CREATE TEMP TABLE a AS
SELECT col1, col2
FROM `project.dataset.table1`
WHERE col1 = 'abc';

WITH
b AS (
  SELECT col2 FROM a
),
c AS (
  SELECT col1 FROM a
)
SELECT b.col2, c.col1
FROM b, c;

DROP TABLE a;
```

## Edge Cases
- Only flag CTEs referenced more than once. A CTE referenced exactly once is fine.
- Materialized views or scripting with `CREATE TEMP TABLE` are the recommended alternatives.
- If the CTE is trivially small (e.g., selecting a few literal values), the impact is negligible.
