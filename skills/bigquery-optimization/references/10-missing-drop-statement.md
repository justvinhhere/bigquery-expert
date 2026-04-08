# MissingDropStatement

**Severity:** Low

## What It Detects
`CREATE TEMP TABLE` statements that do not have a corresponding `DROP TABLE` statement within the same script.

## Why It Matters
Temporary tables still consume storage and are subject to BigQuery's time travel (7 days) and fail-safe (7 days) storage charges. Explicitly dropping temp tables at the end of a script reduces unnecessary storage costs.

## Before
```sql
CREATE TEMP TABLE my_dataset.temp_table (id INT64, name STRING);

SELECT * FROM my_dataset.temp_table;
```

## After
```sql
CREATE TEMP TABLE my_dataset.temp_table (id INT64, name STRING);

SELECT * FROM my_dataset.temp_table;

DROP TABLE my_dataset.temp_table;
```

## Edge Cases
- Only applies within a single script/session context.
- If the script is a stored procedure, dropping temp tables at the end is strongly recommended.
