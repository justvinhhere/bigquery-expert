# JSON Functions

**Impact:** Medium

## What This Covers

BigQuery's native JSON type and extraction functions for querying semi-structured JSON data stored as STRING or native JSON columns.

## Why It Matters

JSON data is common in event logs, API responses, and NoSQL exports. BigQuery's JSON functions let you query this data without pre-flattening, and the native JSON type provides better performance than STRING-based extraction.

## Example

```sql
-- JSON_EXTRACT vs JSON_VALUE vs JSON_QUERY
SELECT
  -- JSON_VALUE: returns a scalar SQL STRING (no quotes)
  JSON_VALUE(payload, '$.user.name') AS user_name,
  -- JSON_QUERY: returns a JSON element (objects/arrays, with quotes)
  JSON_QUERY(payload, '$.user.address') AS address_json,
  -- JSON_EXTRACT: legacy, returns STRING with quotes for strings
  JSON_EXTRACT(payload, '$.user.name') AS user_name_quoted
FROM `project.dataset.events`;

-- Extracting arrays
SELECT
  event_id,
  tag
FROM `project.dataset.events`,
UNNEST(JSON_EXTRACT_ARRAY(payload, '$.tags')) AS tag;

-- Building JSON
SELECT TO_JSON(STRUCT('Alice' AS name, 30 AS age)) AS user_json;
SELECT JSON_OBJECT('name', name, 'email', email) AS user_json
FROM `project.dataset.users`;

-- Native JSON type with dot notation (columns defined as JSON)
SELECT payload.user.name, payload.user.age
FROM `project.dataset.events_native`;
```

## Edge Cases / Pitfalls

- **JSON_VALUE vs JSON_QUERY:** `JSON_VALUE` returns scalars as SQL STRING. `JSON_QUERY` returns JSON elements (objects, arrays). Using the wrong one returns NULL silently.
- **Native JSON type vs STRING:** The native JSON type (column defined as `JSON`) is faster for repeated access and supports dot notation. Prefer it over storing JSON as STRING when you query JSON fields frequently.
- **JSONPath mode:** BigQuery defaults to `lax` mode, which returns NULL for missing paths. Use `strict` mode only when you want errors on missing paths.
- **Performance:** Each `JSON_VALUE` call re-parses the JSON string. For STRING columns accessed multiple times, extract into a CTE first or migrate to native JSON type.
- **Nested arrays:** `JSON_EXTRACT_ARRAY` only works on the top-level array at the given path. For deeply nested arrays, chain multiple extractions.
- **Type coercion:** `JSON_VALUE` always returns STRING. Cast explicitly: `CAST(JSON_VALUE(payload, '$.age') AS INT64)`.
