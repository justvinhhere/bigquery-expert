# Storage Optimization

**Impact:** Medium

## What This Covers
Reducing BigQuery storage costs: active vs. long-term pricing, table expiration, time travel, fail-safe, clones, and streaming buffer overhead.

## Why It Matters
Storage costs accumulate silently. A 100 TB dataset costs $2,000/month in active storage. Proper lifecycle management, expiration policies, and cloning strategies can cut storage costs by 50% or more.

## Best Practice

### Active vs. Long-Term Storage
- **Active**: $0.02/GB/month -- any table/partition modified in the last 90 days.
- **Long-term**: $0.01/GB/month -- auto-applied after 90 days without modification.
- No action needed; BigQuery transitions automatically. Avoid unnecessary writes to old partitions that reset the 90-day clock.

### Table and Partition Expiration
```sql
-- Table expires after 30 days
CREATE TABLE `project.dataset.temp_results`
OPTIONS (expiration_timestamp = TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL 30 DAY))
AS SELECT ...

-- Default partition expiration for all new partitions
ALTER TABLE `project.dataset.events`
SET OPTIONS (partition_expiration_days = 365)
```

### Time Travel and Fail-Safe
| Feature | Duration | Cost | Purpose |
|---------|----------|------|---------|
| Time travel | 2-7 days (configurable) | Billed as active storage | Query/restore past state |
| Fail-safe | 7 days (after time travel) | Billed as active storage | Disaster recovery (Google-managed) |

Reduce time travel to save storage:
```sql
ALTER TABLE `project.dataset.events`
SET OPTIONS (max_time_travel_hours = 48)
```

### Clones vs. Copies
Use `CREATE TABLE ... CLONE` for dev/test (metadata-only, free until divergence). Use `COPY` only when a full physical copy is needed. Monitor with `INFORMATION_SCHEMA.TABLE_STORAGE`.

## Edge Cases / Pitfalls
- Streaming buffer data stays as active storage until flushed.
- DML on a partition resets its 90-day long-term clock.
- Clones grow expensive as they diverge from source via DML.
- Fail-safe storage is always billed and cannot be disabled.
- Dropping a table does not free storage immediately; time travel and fail-safe windows still apply.
