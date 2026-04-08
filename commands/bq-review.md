---
description: Review BigQuery SQL for performance anti-patterns. Pass a file path or paste SQL directly.
argument-hint: "[file_path]"
---

# BigQuery SQL Review

Review the provided BigQuery SQL against all 11 known anti-patterns using the `bigquery-optimization` skill.

## Instructions

1. **Determine the SQL to review:**
   - If a file path argument is provided, read that file.
   - If no argument is provided, look for the most recent BigQuery SQL in the current conversation.
   - If no SQL is found, ask the user to provide a query or file path.

2. **Run the full anti-pattern check** against all 11 patterns:
   - SimpleSelectStar, SemiJoinWithoutAgg, CTEsEvalMultipleTimes, OrderByWithoutLimit,
     StringComparison, LatestRecordWithAnalyticFun, DynamicPredicate, WhereOrder,
     JoinOrder, MissingDropStatement, ConvertTableToTemp.

3. **Output findings** using the standard review format from the bigquery-optimization skill:
   - Group by severity (High, Medium, Low).
   - Include before/after SQL for each finding.
   - Provide a summary count.

4. If no anti-patterns are found, confirm: "No anti-patterns detected. This query follows BigQuery best practices."
