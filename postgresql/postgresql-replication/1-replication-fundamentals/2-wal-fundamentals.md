---
source_course: "postgresql-replication"
source_lesson: "postgresql-replication-wal-fundamentals"
---

# Understanding Write-Ahead Logging

## Introduction

Write-Ahead Logging (WAL) is the foundation of all PostgreSQL replication. Every change to data files is first recorded in the WAL, ensuring durability and enabling both crash recovery and replication.

Understanding WAL architecture and configuration is essential before setting up any form of replication.

## Key Concepts

- **WAL (Write-Ahead Log)**: A sequential log of all modifications made to the database, written before changes reach data files.
- **WAL Segment**: A 16 MB file in the `pg_wal` directory, named with a 24-character hexadecimal string encoding timeline, logical log file, and segment number.
- **LSN (Log Sequence Number)**: A pointer to a position in the WAL stream, used to track replication progress.
- **wal_level**: A server parameter controlling how much information is written to WAL, with three levels: `minimal`, `replica`, and `logical`.

## Real World Context

When a bank processes thousands of transactions per second, every single INSERT and UPDATE must be durable even if the server crashes mid-write. WAL guarantees this by writing changes to a sequential log before applying them. For replication, this same log is shipped to standby servers, which replay it to stay synchronized. Without WAL, neither crash recovery nor replication would be possible.

## Deep Dive

WAL files are stored in the `pg_wal` directory. Each segment file is typically 16 MB and named with a 24-character hexadecimal string:

```
pg_wal/
├── 000000010000000000000001
├── 000000010000000000000002
├── 000000010000000000000003
└── archive_status/
```

The naming convention encodes: `<timeline><logical_log_file><segment>`.

You can inspect the current WAL position and settings with these queries:

```sql
-- Check current WAL position
SELECT pg_current_wal_lsn();

-- View WAL settings
SHOW wal_level;             -- minimal, replica, or logical
SHOW max_wal_senders;       -- max concurrent streaming connections
SHOW max_replication_slots;  -- max replication slots
```

These queries help you verify that the server is configured correctly before attempting to set up replication.

The three WAL levels control what information is recorded:

| Level | Use Case | Overhead |
|-------|----------|----------|
| minimal | Standalone server, no replication | Lowest |
| replica | Physical streaming replication | Medium |
| logical | Logical replication | Highest |

Changing `wal_level` requires a server restart. This configuration snippet shows the essential WAL settings for replication:

```sql
-- postgresql.conf settings for replication
wal_level = replica           -- minimum for streaming replication
max_wal_senders = 10          -- number of concurrent connections
max_replication_slots = 10    -- prevent WAL removal for slow standbys
wal_keep_size = 1GB           -- keep WAL for catching up standbys
```

These settings ensure the primary server generates sufficient WAL detail and retains segments long enough for standbys to consume them.

### max_active_replication_origins (PostgreSQL 18)

PostgreSQL 18 introduces the `max_active_replication_origins` parameter (default 10, requires restart). Previously, the number of active replication origins was controlled by `max_replication_slots`. Now it is an independent setting, giving you finer-grained control over resource allocation:

```sql
-- postgresql.conf (PG18+)
max_active_replication_origins = 20  -- independent of max_replication_slots
```

This is useful when you have many logical subscriptions but want to keep `max_replication_slots` lower for physical replication.

## Common Pitfalls

- **Leaving wal_level at minimal**: If `wal_level` is `minimal`, no replication is possible. Always verify the setting before configuring standbys.
- **Setting max_wal_senders too low**: Each standby and each `pg_basebackup` consumes a WAL sender slot. Running out causes connection failures.
- **Forgetting that wal_level changes require restart**: Unlike many PostgreSQL parameters, `wal_level` cannot be changed with a simple reload.

## Best Practices

- Set `wal_level = logical` if you anticipate ever using logical replication; it is a superset of `replica` and avoids a future restart.
- Always configure `max_replication_slots` to at least the number of standbys plus a margin for backups and monitoring tools.
- Monitor WAL generation rate and disk usage in `pg_wal` to prevent disk exhaustion.

## Summary

- WAL is the sequential log underlying all PostgreSQL replication and crash recovery.
- WAL segments are 16 MB files stored in `pg_wal`, named by timeline and position.
- The `wal_level` parameter controls replication capability: `minimal` (none), `replica` (physical), `logical` (both).
- PostgreSQL 18 adds `max_active_replication_origins` as an independent parameter for finer resource control.
- Changing `wal_level` requires a server restart.

## Code Examples

**WAL Configuration for Replication**

```sql
-- postgresql.conf settings for replication
wal_level = replica           -- minimum for streaming replication
max_wal_senders = 10          -- number of concurrent connections
max_replication_slots = 10    -- prevent WAL removal for slow standbys
wal_keep_size = 1GB           -- keep WAL for catching up standbys
```


## Resources

- [WAL Configuration](https://www.postgresql.org/docs/current/wal-configuration.html) — undefined

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*