# SemiJoinWithoutAgg

**Severity:** Medium

## What It Detects
`IN` or `NOT IN` subqueries where the subquery does not use `DISTINCT` or `GROUP BY`. This means the subquery may return duplicate values, forcing BigQuery to process redundant comparisons.

## Why It Matters
Without deduplication, the subquery can return millions of duplicate values. BigQuery must process each one in the semi-join, wasting compute resources and increasing slot time.

## Before
```sql
SELECT t1.col1
FROM `project.dataset.table1` t1
WHERE t1.col2 NOT IN (SELECT col2 FROM `project.dataset.table2`)
```

## After
```sql
SELECT t1.col1
FROM `project.dataset.table1` t1
WHERE t1.col2 NOT IN (SELECT DISTINCT col2 FROM `project.dataset.table2`)
```

## Alternative Fix (using GROUP BY)
```sql
SELECT t1.col1
FROM `project.dataset.table1` t1
WHERE t1.col2 NOT IN (SELECT col2 FROM `project.dataset.table2` GROUP BY col2)
```

## Edge Cases
- If the subquery column is already unique (e.g., a primary key), adding `DISTINCT` has minimal impact but is still good practice.
- Applies to both `IN` and `NOT IN` predicates.
