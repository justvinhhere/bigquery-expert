# SQL Dialect Conversion to BigQuery

**Impact:** High

## What This Covers
Mapping table for converting SQL from PostgreSQL, MySQL, Snowflake, Redshift, and SQL Server to idiomatic BigQuery SQL.

## Why It Matters
Direct copy-paste from other databases produces syntax errors or silently wrong results. Correct conversion requires understanding semantic differences, not just syntax swaps.

## Best Practice

### Core syntax mappings
| Source (dialect) | BigQuery equivalent |
|-----------------|---------------------|
| `ILIKE` (PG, SF) | `LOWER(col) LIKE LOWER('pattern')` |
| `NVL(a, b)` (Oracle, SF) | `IFNULL(a, b)` |
| `DATEADD(day, 7, col)` (SF, RS) | `DATE_ADD(col, INTERVAL 7 DAY)` |
| `DATEDIFF(day, a, b)` (SF, RS, SS) | `DATE_DIFF(b, a, DAY)` -- args swapped |
| `TOP 100` (SS) | `LIMIT 100` at end of query |
| `col::INT` (PG, SF) | `CAST(col AS INT64)` |
| `GETDATE()`/`NOW()`/`SYSDATE` | `CURRENT_TIMESTAMP()` |
| `LEN(s)` (SS, RS) | `LENGTH(s)` |
| `CHARINDEX(sub, s)` (SS) | `STRPOS(s, sub)` -- args swapped |
| `CONVERT(type, expr)` (SS) | `CAST(expr AS type)` |

### QUALIFY, set operations, dates
- **QUALIFY:** BigQuery supports it natively. Preserve from Snowflake/Databricks as-is.
- **EXCEPT / MINUS:** BigQuery requires the explicit `DISTINCT` keyword -- plain `EXCEPT` is a syntax error in GoogleSQL. Convert `EXCEPT` (PG) and `MINUS` (Oracle) to `EXCEPT DISTINCT`. Similarly, `INTERSECT` → `INTERSECT DISTINCT`.
- **Date literals:** `DATE '2024-01-15'`. **TZ timestamps:** `TIMESTAMP('2024-01-15 10:00:00', 'America/New_York')`.
- **Epoch:** `UNIX_SECONDS(ts)`. **Truncation:** `DATE_TRUNC(d, MONTH)`, `TIMESTAMP_TRUNC(ts, HOUR)`.

### NULL handling and SAFE_ prefix
- `IS DISTINCT FROM` (PG) -- supported natively in BigQuery GoogleSQL. Preserve as-is.
- `SAFE_DIVIDE`, `SAFE_CAST` return `NULL` instead of errors. Use proactively.

### STRUCT/ARRAY and MERGE
BigQuery has native `STRUCT`/`ARRAY` with no equivalent in most dialects. Convert JSON/array-of-rows patterns:
```sql
SELECT user_id, ARRAY_AGG(STRUCT(event_name, event_time)) AS events
FROM `project.dataset.raw_events` GROUP BY user_id
```
**MERGE:** Same syntax as SQL Server/Snowflake but no `OUTPUT` clause support.

## Edge Cases / Pitfalls
- **Recursive CTEs:** Supported natively via `WITH RECURSIVE` (up to 500 iterations). For Oracle `CONNECT BY` patterns that exceed this depth, use `LOOP`/`WHILE` in a script.
- **Auto-increment:** No `SERIAL`/`IDENTITY`. Use `GENERATE_UUID()` or `ROW_NUMBER()`.
- **Types:** `INT64` not `INT`, `FLOAT64` not `FLOAT`. `BOOLEAN` is accepted as alias for `BOOL`.
- **Case sensitivity:** BQ strings are case-sensitive. Migrated queries from case-insensitive collations need `LOWER()`.
- **Division:** Integer division returns `FLOAT64`. Use `DIV(a, b)` for integer result.
