---
description: Optimize BigQuery SQL by fixing all detected anti-patterns. Pass a file path or paste SQL directly.
argument-hint: "[file_path]"
---

# BigQuery SQL Optimize

Rewrite the provided BigQuery SQL with all detected anti-patterns fixed, using the `bigquery-optimization` skill.

## Instructions

1. **Determine the SQL to optimize:**
   - If a file path argument is provided, read that file.
   - If no argument is provided, look for the most recent BigQuery SQL in the current conversation.
   - If no SQL is found, ask the user to provide a query or file path.

2. **Detect all anti-patterns** by running the full 11-pattern check.

3. **Apply all fixes** to produce an optimized version of the query:
   - Preserve the original query semantics exactly -- the optimized query must return the same data.
   - Apply fixes in this order: SimpleSelectStar, SemiJoinWithoutAgg, DynamicPredicate,
     OrderByWithoutLimit, StringComparison, CTEsEvalMultipleTimes, LatestRecordWithAnalyticFun,
     WhereOrder, MissingDropStatement, ConvertTableToTemp.

4. **Output the result:**

```
## Optimized Query

(fully rewritten SQL)

## Changes Applied

1. **PatternName**: What was changed and why.
2. **PatternName**: What was changed and why.

## Summary
X anti-pattern(s) fixed. The optimized query preserves the original semantics.
```

5. If no anti-patterns are found, return the original query and confirm it already follows best practices.
