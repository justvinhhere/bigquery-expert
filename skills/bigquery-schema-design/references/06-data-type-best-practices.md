# Data Type Best Practices

**Impact:** Medium

## What This Covers
Choosing optimal BigQuery data types: temporal, numeric, identifier, and JSON types.

## Why It Matters
Wrong types waste storage, slow comparisons, and cause correctness bugs. STRING for numeric IDs wastes ~3x storage. FLOAT64 for money introduces rounding errors.

## Best Practice

**Temporal types:**
```sql
event_ts TIMESTAMP,       -- absolute UTC point in time (8 bytes) -- use for events/logs
appointment_dt DATETIME,  -- no timezone, calendar semantics (8 bytes)
birth_date DATE,          -- date only (4 bytes)
```
Use TIMESTAMP for event data. Use DATETIME only when timezone is irrelevant.

**Numeric types:**
```sql
order_count INT64,        -- integers, counters (8 bytes)
price NUMERIC,            -- exact decimal, 38 digits / 9 decimal (16 bytes) -- use for money
ml_score FLOAT64,         -- approximate double-precision (8 bytes)
precise_val BIGNUMERIC    -- 76 total digits / 38 decimal places (32 bytes) -- rarely needed
```

**Identifier types:**
```sql
user_id INT64,            -- numeric IDs: faster joins, smaller storage
session_id STRING         -- UUIDs/alphanumeric: use STRING, avoid casting
```

**JSON type vs STRING:**
```sql
metadata JSON,            -- validates on insert, native JSON_VALUE() support
raw_payload STRING        -- opaque blobs you rarely parse
```

**Storage sizes:** BOOL=1B, INT64/FLOAT64/TIMESTAMP/DATETIME=8B, DATE/TIME=4B, NUMERIC=16B, BIGNUMERIC=32B, STRING/JSON=2B+length.

## Edge Cases / Pitfalls
- FLOAT64 cannot represent 0.1 exactly. Never use for currency or balances.
- NUMERIC caps at 9 decimal places. Use BIGNUMERIC (38 decimals) at 2x storage cost.
- Casting STRING to INT64 per-row at query time is expensive. Fix the source schema.
- TIMESTAMP arithmetic returns INTERVAL. Use `TIMESTAMP_DIFF()` for numeric results.
- JSON type (added 2022) outperforms `JSON_EXTRACT` on STRING columns. Prefer it for structured metadata.
