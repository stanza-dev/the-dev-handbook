---
source_course: "postgresql-replication"
source_lesson: "postgresql-replication-replication-slots-deep-dive"
---

# Replication Slots and WAL Retention

## Introduction

Replication slots are a critical mechanism that prevents the primary server from discarding WAL segments that standbys or subscribers still need. Without them, a slow or disconnected replica can fall behind and lose data.

This lesson covers how replication slots work, how to manage them, and the new PostgreSQL 18 parameter for automatic cleanup of inactive slots.

## Key Concepts

- **Replication Slot**: A server-side object that tracks the replication progress of a consumer, preventing WAL removal past the consumer's confirmed position.
- **Physical Replication Slot**: Used by streaming replication standbys to ensure WAL retention.
- **Logical Replication Slot**: Used by logical replication subscribers, also tracks catalog information needed for decoding.
- **idle_replication_slot_timeout**: A new PostgreSQL 18 parameter that automatically invalidates inactive replication slots.

## Real World Context

Imagine a standby server goes down for maintenance over a weekend. Without a replication slot, the primary may recycle the WAL segments the standby needs, forcing a full re-sync when it returns. With a replication slot, the primary retains all necessary WAL, but this creates a different risk: if the standby never comes back, WAL accumulates indefinitely and can fill the disk. PostgreSQL 18 solves this with automatic slot invalidation.

## Deep Dive

### Creating and Managing Slots

Physical and logical replication slots are created differently:

```sql
-- Create a physical replication slot
SELECT pg_create_physical_replication_slot('standby1_slot');

-- Create a logical replication slot
SELECT pg_create_logical_replication_slot('analytics_slot', 'pgoutput');

-- View all replication slots
SELECT slot_name, slot_type, active, restart_lsn,
       pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn) AS retained_bytes
FROM pg_replication_slots;
```

This query shows each slot, whether it is active, and how many bytes of WAL it is retaining.

### Monitoring Slot Health

Inactive slots are the most common cause of WAL disk bloat:

```sql
-- Find inactive slots retaining significant WAL
SELECT slot_name, slot_type, active,
       pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn)) AS retained_wal
FROM pg_replication_slots
WHERE NOT active
ORDER BY pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn) DESC;
```

If you find an abandoned slot retaining gigabytes of WAL, you should drop it:

```sql
-- Drop an unused slot (WARNING: consumer will need full re-sync)
SELECT pg_drop_replication_slot('abandoned_slot');
```

Dropping the slot releases the retained WAL for recycling.

### idle_replication_slot_timeout (PostgreSQL 18)

PostgreSQL 18 introduces `idle_replication_slot_timeout` to automatically invalidate inactive replication slots, preventing WAL bloat without manual intervention:

```sql
-- postgresql.conf (PG18+)
idle_replication_slot_timeout = '3d'  -- invalidate after 3 days of inactivity
-- Default: 0 (disabled)
```

When enabled, PostgreSQL checks for inactive slots during checkpoints and invalidates any that have been idle longer than the timeout. The invalidation reason is recorded in `pg_replication_slots`:

```sql
-- Check for invalidated slots
SELECT slot_name, active, invalidation_reason
FROM pg_replication_slots
WHERE invalidation_reason IS NOT NULL;
```

This prevents the all-too-common scenario of a forgotten slot slowly consuming all available disk space.

## Common Pitfalls

- **Forgetting to drop unused slots**: An inactive replication slot retains WAL indefinitely, which can fill the primary's disk and cause an outage.
- **Creating slots without monitoring**: Always set up alerting on WAL retention per slot so you catch problems before disk exhaustion.
- **Using wal_keep_size instead of slots**: The `wal_keep_size` parameter is a blunt instrument that does not guarantee retention for a specific consumer; slots provide precise tracking.

## Best Practices

- Always use replication slots for both physical and logical replication to guarantee WAL availability.
- On PostgreSQL 18+, enable `idle_replication_slot_timeout` to automatically clean up abandoned slots.
- Monitor the `pg_replication_slots` view and alert when retained WAL exceeds a threshold.

## Summary

- Replication slots prevent the primary from discarding WAL that replicas still need.
- Physical slots track streaming replication consumers; logical slots also track catalog state for decoding.
- Inactive slots cause WAL bloat and potential disk exhaustion if not monitored.
- PostgreSQL 18 adds `idle_replication_slot_timeout` to automatically invalidate idle slots during checkpoints.
- Always monitor slot health and set up alerts for retained WAL size.

## Code Examples

**Replication Slot Management**

```sql
-- Create physical slot for standby
SELECT pg_create_physical_replication_slot('standby1_slot');

-- Monitor retained WAL per slot
SELECT slot_name, slot_type, active,
       pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn)) AS retained_wal
FROM pg_replication_slots;

-- PG18: Auto-invalidate idle slots
-- ALTER SYSTEM SET idle_replication_slot_timeout = '3d';
```


## Resources

- [Replication Slots](https://www.postgresql.org/docs/current/warm-standby.html#STREAMING-REPLICATION-SLOTS) — undefined

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*