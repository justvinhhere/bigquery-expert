# StringComparison

**Severity:** Low

## What It Detects
Use of `REGEXP_CONTAINS()` with simple wildcard patterns like `'.*value.*'` that could be replaced with the `LIKE` operator.

## Why It Matters
Regular expression evaluation is computationally more expensive than simple string matching. `LIKE '%value%'` achieves the same result as `REGEXP_CONTAINS(col, '.*value.*')` with less overhead.

## Before
```sql
SELECT dim1
FROM `dataset.table`
WHERE REGEXP_CONTAINS(dim1, '.*test.*')
```

## After
```sql
SELECT dim1
FROM `dataset.table`
WHERE dim1 LIKE '%test%'
```

## Edge Cases
- Only flag `REGEXP_CONTAINS` with simple `.*X.*` patterns. Complex regex patterns (character classes, alternation, anchors) legitimately need `REGEXP_CONTAINS`.
- `REGEXP_CONTAINS(col, 'exact_value')` without wildcards can be replaced with `col LIKE '%exact_value%'` or even `col = 'exact_value'` depending on intent.
