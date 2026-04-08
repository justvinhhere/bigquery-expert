# ConvertTableToTemp

**Severity:** Low

## What It Detects
A persistent table (`CREATE TABLE`) that is created and then dropped (`DROP TABLE`) within the same script. This indicates the table is only used temporarily but incurs persistent storage costs.

## Why It Matters
Persistent tables incur time travel (2--7 days, configurable at dataset level) and fail-safe (7 days) storage costs even after being dropped. Under the physical storage billing model, these appear as separate charges. Converting to `CREATE TEMP TABLE` avoids these charges entirely since temp tables are not subject to time travel or fail-safe.

## Before
```sql
CREATE TABLE `my_dataset.staging_data` (id INT64, name STRING);
-- ... processing ...
DROP TABLE `my_dataset.staging_data`;
```

## After
```sql
CREATE TEMP TABLE staging_data (id INT64, name STRING);
-- ... processing ...
DROP TABLE staging_data;
```

## Edge Cases
- Only flag when both `CREATE TABLE` and `DROP TABLE` for the same table exist in the same script.
- If the table is accessed by other sessions/queries between creation and dropping, it cannot be converted to TEMP.
- `CREATE OR REPLACE TABLE` followed by `DROP TABLE` should also be flagged.
