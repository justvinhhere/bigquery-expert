# BigQuery Scripting

**Impact:** High

## What This Covers

Multi-statement scripts with variables, control flow, loops, exception handling, procedures, and transactions in BigQuery.

## Why It Matters

Scripting enables complex ETL workflows, conditional logic, and error handling directly in BigQuery without external orchestrators. Reduces data movement and simplifies pipeline architecture.

## Example

```sql
-- Variables and control flow
DECLARE target_date DATE DEFAULT CURRENT_DATE() - 1;
DECLARE row_count INT64;

BEGIN
  -- Conditional processing
  IF EXTRACT(DAYOFWEEK FROM target_date) = 1 THEN
    -- Sunday: full refresh
    CREATE OR REPLACE TABLE `project.dataset.weekly_agg` AS
    SELECT region, SUM(revenue) AS total_revenue
    FROM `project.dataset.sales`
    WHERE sale_date BETWEEN DATE_TRUNC(target_date, WEEK) AND target_date
    GROUP BY region;
  ELSE
    -- Weekday: incremental append
    INSERT INTO `project.dataset.daily_agg`
    SELECT region, SUM(revenue) AS total_revenue, target_date
    FROM `project.dataset.sales`
    WHERE sale_date = target_date
    GROUP BY region;
  END IF;

  SET row_count = @@row_count;

EXCEPTION WHEN ERROR THEN
  -- Log the error
  INSERT INTO `project.dataset.error_log`
  VALUES (CURRENT_TIMESTAMP(), @@error.message, @@error.statement_text);
END;

-- LOOP with BREAK
DECLARE i INT64 DEFAULT 0;
LOOP
  SET i = i + 1;
  IF i > 10 THEN BREAK; END IF;
END LOOP;
```

## Edge Cases / Pitfalls

- **System variables:** `@@row_count` resets after every statement. Capture it immediately with `SET` if needed later.
- **Transactions:** Use `BEGIN TRANSACTION...COMMIT` for explicit multi-statement transactions that can span multiple tables. If you mutate a table within a transaction, no other concurrent transaction can mutate that same table until the first commits or rolls back.
- **Script size:** A single script can contain up to 1,000 statements. For larger workflows, split into multiple scripts or use stored procedures.
- **EXECUTE IMMEDIATE:** Useful for dynamic SQL but prevents query caching. Use parameterized queries (`USING`) when possible.
- **Error handling:** `BEGIN...EXCEPTION...END` catches runtime errors but not syntax errors. Syntax errors abort the script before execution.
- **Slot consumption:** Each statement in a script consumes slots sequentially. Long scripts can hold reservations. Prefer batch operations over row-by-row loops.
