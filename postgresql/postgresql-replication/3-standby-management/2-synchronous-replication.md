---
source_course: "postgresql-replication"
source_lesson: "postgresql-replication-synchronous-replication"
---

# Synchronous Replication

## Introduction

Synchronous replication guarantees that transactions are replicated to at least one standby before the primary confirms the commit. This provides zero data loss in case of primary failure, at the cost of increased commit latency.

This lesson covers the different synchronous commit levels, quorum-based configurations, and per-transaction overrides for performance tuning.

## Key Concepts

- **synchronous_commit**: Controls how much replication confirmation the primary waits for before reporting a commit as successful.
- **synchronous_standby_names**: Specifies which standbys participate in synchronous replication, using FIRST or ANY syntax for quorum configurations.
- **remote_apply**: The strictest synchronous commit level, ensuring the transaction is visible on the standby before the primary confirms.
- **Quorum commit**: Requires confirmation from a specified number of standbys (e.g., ANY 2 of 3), providing fault tolerance among replicas.

## Real World Context

A payment processing system cannot tolerate any data loss. With `synchronous_commit = on` and `synchronous_standby_names = 'ANY 2 (dc1, dc2, dc3)'`, every committed transaction is confirmed by at least two data centers before the application receives success. If one data center goes offline, the remaining two continue providing synchronous replication without interruption.

## Deep Dive

PostgreSQL provides five synchronous commit levels, each offering a different trade-off:

| Level | Durability | Performance |
|-------|------------|-------------|
| off | Lowest | Highest |
| local | Primary only | High |
| remote_write | Standby received | Medium |
| on | Standby flushed | Lower |
| remote_apply | Standby applied | Lowest |

Configure synchronous replication on the primary:

```sql
-- postgresql.conf
synchronous_commit = on
synchronous_standby_names = 'standby1'  -- Single sync standby

-- Multiple standbys with priority
synchronous_standby_names = 'FIRST 2 (standby1, standby2, standby3)'

-- Quorum-based: any 2 must confirm
synchronous_standby_names = 'ANY 2 (standby1, standby2, standby3)'
```

The FIRST syntax uses priority ordering (standby1 is preferred), while ANY syntax accepts confirmation from any N standbys in the list.

Standbys identify themselves via `application_name` in the connection string:

```sql
-- On standby postgresql.conf or connection string
primary_conninfo = 'host=primary application_name=standby1'
```

This name must match what appears in `synchronous_standby_names`.

Monitor synchronous standby status from the primary:

```sql
-- Check sync status
SELECT application_name, client_addr, sync_state, sync_priority
FROM pg_stat_replication;

-- sync_state values:
-- 'sync' = synchronous standby
-- 'async' = asynchronous standby
-- 'potential' = could become sync if current fails
```

The `sync_state` column shows which standbys are currently synchronous.

For bulk operations where latency matters more than per-transaction durability, use per-transaction overrides:

```sql
-- Use async for bulk loads
SET synchronous_commit = off;
INSERT INTO logs SELECT generate_series(1, 1000000);
SET synchronous_commit = on;
```

This avoids the replication round-trip for each row in the bulk load while keeping synchronous mode for regular transactions.

## Common Pitfalls

- **All synchronous standbys going offline**: If all named standbys become unavailable, every write on the primary will hang indefinitely waiting for synchronous confirmation. Monitor standby health carefully.
- **Confusing remote_write with on**: `remote_write` only confirms the standby received the data in its OS buffer; `on` confirms it was flushed to disk. Only `on` or `remote_apply` survive a standby crash.
- **Not using application_name**: Without a matching `application_name`, the standby will not be recognized as a synchronous participant.

## Best Practices

- Use `ANY 2` with three or more standbys so that one standby failure does not block writes.
- Use `remote_apply` only when standby queries must see committed data immediately; `on` is sufficient for data durability.
- Override `synchronous_commit` per-transaction for batch operations that do not require individual transaction durability.

## Summary

- Synchronous replication guarantees transactions are replicated before commit confirmation, providing zero data loss.
- Five commit levels offer a spectrum from highest performance (off) to highest durability (remote_apply).
- Use FIRST for priority-based failover or ANY for quorum-based fault tolerance.
- Per-transaction overrides allow performance tuning for batch operations.
- Monitor sync status via `pg_stat_replication` and ensure at least one sync standby is always available.

## Code Examples

**Synchronous Quorum Configuration**

```sql
-- Primary requires any 2 standbys to confirm
synchronous_commit = on
synchronous_standby_names = 'ANY 2 (dc1_standby, dc2_standby, dc3_standby)'

-- Query to check sync status
SELECT application_name, state, sync_state,
       sent_lsn, write_lsn, flush_lsn, replay_lsn
FROM pg_stat_replication;
```


## Resources

- [Replication Configuration](https://www.postgresql.org/docs/current/runtime-config-replication.html) — undefined

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*