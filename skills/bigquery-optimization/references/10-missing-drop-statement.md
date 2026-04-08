# MissingDropStatement

**Severity:** Low

## What It Detects
`CREATE TEMP TABLE` statements that do not have a corresponding `DROP TABLE` statement within the same script.

## Why It Matters
Temporary tables consume active storage for the duration of the session and are automatically destroyed when the session ends. While they are not subject to time travel or fail-safe storage charges, explicitly dropping temp tables at the end of a script reclaims resources immediately in long-running sessions and keeps scripts clean.

## Before
```sql
CREATE TEMP TABLE temp_table (id INT64, name STRING);

SELECT * FROM temp_table;
```

## After
```sql
CREATE TEMP TABLE temp_table (id INT64, name STRING);

SELECT * FROM temp_table;

DROP TABLE temp_table;
```

## Edge Cases
- Only applies within a single script/session context.
- If the script is a stored procedure, dropping temp tables at the end is strongly recommended.
