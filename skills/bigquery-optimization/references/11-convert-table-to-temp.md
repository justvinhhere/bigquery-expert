# ConvertTableToTemp

**Severity:** Low

## What It Detects
A persistent table (`CREATE TABLE`) that is created and then dropped (`DROP TABLE`) within the same script. This indicates the table is only used temporarily but incurs persistent storage costs.

## Why It Matters
Persistent tables incur time travel (7 days) and fail-safe (7 days) storage costs even after being dropped. Converting to `CREATE TEMP TABLE` avoids these charges entirely.

## Before
```sql
CREATE TABLE `my_dataset.staging_data` (id INT64, name STRING);
-- ... processing ...
DROP TABLE `my_dataset.staging_data`;
```

## After
```sql
CREATE TEMP TABLE `my_dataset.staging_data` (id INT64, name STRING);
-- ... processing ...
DROP TABLE `my_dataset.staging_data`;
```

## Edge Cases
- Only flag when both `CREATE TABLE` and `DROP TABLE` for the same table exist in the same script.
- If the table is accessed by other sessions/queries between creation and dropping, it cannot be converted to TEMP.
- `CREATE OR REPLACE TABLE` followed by `DROP TABLE` should also be flagged.
