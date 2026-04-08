# DynamicPredicate

**Severity:** Medium

## What It Detects
Subqueries used directly inside `WHERE` clause predicates. These "dynamic predicates" are recomputed during query execution, preventing query optimization.

## Why It Matters
BigQuery cannot optimize a filter that depends on a subquery result computed at runtime. Extracting the subquery result into a variable (using `DECLARE`) pre-computes the value before the main query executes, improving optimization opportunities and avoiding repeated subquery evaluation. Note: `DECLARE` variables are resolved at runtime and may not guarantee partition pruning the same way literal constants do, but they consistently improve query plan efficiency.

## Before (Single-Value)
```sql
SELECT col1
FROM `dataset.table1`
WHERE col2 = (SELECT MAX(col3) FROM `dataset.table2`)
```

## After (Single-Value)
```sql
DECLARE max_val DEFAULT (SELECT MAX(col3) FROM `dataset.table2`);

SELECT col1
FROM `dataset.table1`
WHERE col2 = max_val
```

## Before (Multi-Value with IN)
```sql
SELECT descript
FROM `dataset.tbl2`
WHERE unique_key IN (SELECT sk FROM `dataset.tbl1`)
```

## After (Multi-Value with IN)
```sql
DECLARE keys ARRAY<STRING> DEFAULT (SELECT ARRAY_AGG(sk) FROM `dataset.tbl1`);

SELECT descript
FROM `dataset.tbl2`
WHERE unique_key IN UNNEST(keys)
```

## Edge Cases
- For single-value predicates (`=`, `>`, `<`), use a scalar `DECLARE` variable.
- For multi-value predicates (`IN`), use `DECLARE var ARRAY<type>` with `ARRAY_AGG()`, then `IN UNNEST(var)`.
- If the subquery is correlated (references outer query columns), it cannot be extracted to a variable.
- Small subqueries with very fast execution may have negligible impact.
