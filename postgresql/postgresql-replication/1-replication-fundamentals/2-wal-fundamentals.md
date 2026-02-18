---
source_course: "postgresql-replication"
source_lesson: "wal-fundamentals"
---

# Understanding Write-Ahead Logging

Write-Ahead Logging (WAL) is the foundation of all PostgreSQL replication. Understanding WAL is essential for configuring and troubleshooting replication.

## What is WAL?

WAL is PostgreSQL's mechanism for ensuring data durability. Before any change is written to data files, it's first recorded in the WAL. This provides:

1. **Crash Recovery**: After a crash, PostgreSQL replays WAL to restore consistency
2. **Replication**: Standbys replay WAL records to stay synchronized
3. **Point-in-Time Recovery**: WAL archives enable restoring to any point in time

## WAL Architecture

WAL files are stored in the `pg_wal` directory (formerly `pg_xlog`). Each segment file is typically 16 MB and named with a 24-character hexadecimal string:

```
pg_wal/
├── 000000010000000000000001
├── 000000010000000000000002
├── 000000010000000000000003
└── archive_status/
```

The naming convention encodes: `<timeline><logical_log_file><segment>`

## Key WAL Settings

```sql
-- Check current WAL position
SELECT pg_current_wal_lsn();

-- View WAL settings
SHOW wal_level;           -- minimal, replica, or logical
SHOW max_wal_senders;     -- max concurrent streaming connections
SHOW max_replication_slots; -- max replication slots
```

## WAL Levels Explained

| Level | Use Case | Overhead |
|-------|----------|----------|
| minimal | Standalone server, no replication | Lowest |
| replica | Physical streaming replication | Medium |
| logical | Logical replication | Highest |

**Important:** Changing `wal_level` requires a server restart.

## Code Examples

```undefined
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