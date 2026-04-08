# OrderByWithoutLimit

**Severity:** Medium

## What It Detects
An `ORDER BY` clause at the outermost query level without a corresponding `LIMIT` clause.

**Not flagged:** `ORDER BY` inside subqueries, CTEs, or window function `OVER()` clauses.

## Why It Matters
Sorting large result sets is computationally expensive. Without `LIMIT`, the entire result set must be sorted in memory. For large tables, this can cause memory spills and dramatically increase query time and slot usage.

## Before
```sql
SELECT t.dim1, t.dim2, t.metric1
FROM `dataset.table` t
ORDER BY t.metric1 DESC
```

## After
```sql
SELECT t.dim1, t.dim2, t.metric1
FROM `dataset.table` t
ORDER BY t.metric1 DESC
LIMIT 1000
```

## Edge Cases
- If `ORDER BY` is required for correctness (e.g., producing a deterministically ordered export), this is acceptable -- but add a comment explaining why.
- `ORDER BY` inside a subquery used by a window function or `ARRAY_AGG` is fine and should not be flagged.
