---
source_course: "postgresql-replication"
source_lesson: "postgresql-replication-streaming-setup"
---

# Configuring Streaming Replication

## Introduction

Streaming replication sends WAL records directly from the primary to the standby over a TCP connection, providing near-real-time replication with sub-second lag in most configurations.

This lesson walks through the complete configuration of both primary and standby servers, including replication slot management and lag monitoring.

## Key Concepts

- **primary_conninfo**: A connection string on the standby that specifies how to connect to the primary server for streaming WAL.
- **standby.signal**: A file in the data directory that tells PostgreSQL to start in standby mode rather than as a primary.
- **Replication Slot**: A server-side object that tracks a consumer's position and prevents the primary from discarding needed WAL.
- **pg_stat_replication**: A system view on the primary showing all connected standbys and their replication progress.

## Real World Context

A content management platform needs a read replica in each geographic region to serve local users with low latency. Each regional standby connects to the central primary via streaming replication, maintaining a near-identical copy of the database. The operations team monitors `pg_stat_replication` to ensure each standby stays within acceptable lag, alerting if any replica falls more than 10 seconds behind.

## Deep Dive

Start with the primary server configuration in `postgresql.conf`:

```sql
-- postgresql.conf on primary
wal_level = replica
max_wal_senders = 10
max_replication_slots = 10
hot_standby = on

-- Recommended for reliability
synchronous_commit = on
wal_keep_size = 1GB
```

These settings enable WAL shipping, allocate connection slots for standbys, and keep WAL segments as a safety net.

Replication slots prevent the primary from removing WAL files that the standby still needs:

```sql
-- Create a physical replication slot
SELECT pg_create_physical_replication_slot('standby_slot');

-- View existing slots
SELECT slot_name, slot_type, active, restart_lsn
FROM pg_replication_slots;

-- Drop unused slot (WARNING: can cause WAL bloat if forgotten)
SELECT pg_drop_replication_slot('old_slot');
```

Always drop slots for decommissioned standbys to prevent WAL accumulation.

On the standby, create `standby.signal` in the data directory and configure the connection:

```sql
-- postgresql.auto.conf or postgresql.conf on standby
primary_conninfo = 'host=primary port=5432 user=replicator password=secret'
primary_slot_name = 'standby_slot'
hot_standby = on
```

These three settings tell the standby where to connect, which slot to use, and to accept read-only queries.

Monitor replication health from both sides:

```sql
-- On primary: view connected standbys
SELECT client_addr, state, sent_lsn, write_lsn, flush_lsn, replay_lsn,
       pg_wal_lsn_diff(sent_lsn, replay_lsn) AS lag_bytes
FROM pg_stat_replication;

-- On standby: verify recovery mode
SELECT pg_is_in_recovery();  -- Should return true
SELECT pg_last_wal_receive_lsn();
SELECT pg_last_wal_replay_lsn();

-- Calculate lag in seconds
SELECT EXTRACT(EPOCH FROM (now() - pg_last_xact_replay_timestamp()));
```

The `lag_bytes` column shows how far behind the standby is in bytes, while the time-based calculation shows lag in seconds.

## Common Pitfalls

- **Not setting primary_slot_name**: Without a slot, the primary may recycle WAL before the standby consumes it, especially during network outages.
- **Forgetting to drop slots for removed standbys**: An orphaned slot retains WAL indefinitely and will eventually fill the disk.
- **Confusing sent_lsn with replay_lsn**: The standby may have received WAL (write_lsn) but not yet replayed it (replay_lsn). True replication completeness requires checking replay_lsn.

## Best Practices

- Always use replication slots for reliable WAL retention.
- Monitor `pg_stat_replication` on the primary and set alerts for lag exceeding your RPO threshold.
- Use `application_name` in `primary_conninfo` to identify standbys in monitoring views.

## Summary

- Streaming replication requires `wal_level = replica` and `max_wal_senders` on the primary, plus `standby.signal` and `primary_conninfo` on the standby.
- Replication slots prevent WAL removal but must be dropped when standbys are decommissioned.
- Monitor replication via `pg_stat_replication` on the primary and `pg_is_in_recovery()` on the standby.
- Calculate lag in both bytes and seconds for a complete picture of replication health.

## Code Examples

**Complete Streaming Replication Setup**

```sql
-- On PRIMARY: Create replication user and slot
CREATE ROLE replicator WITH REPLICATION LOGIN PASSWORD 'secure_password';
SELECT pg_create_physical_replication_slot('standby1_slot');

-- On STANDBY: After pg_basebackup -R, verify connection
-- Check postgresql.auto.conf contains:
-- primary_conninfo = 'host=primary port=5432 user=replicator password=...'
-- primary_slot_name = 'standby1_slot'
```


## Resources

- [Log-Shipping Standby Servers](https://www.postgresql.org/docs/current/warm-standby.html) — undefined

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*