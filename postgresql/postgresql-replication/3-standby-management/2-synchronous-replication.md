---
source_course: "postgresql-replication"
source_lesson: "synchronous-replication"
---

# Synchronous Replication

Synchronous replication guarantees that transactions are replicated before the primary confirms the commit, providing zero data loss in case of primary failure.

## Synchronous Commit Levels

| Level | Durability | Performance |
|-------|------------|-------------|
| off | Lowest | Highest |
| local | Primary only | High |
| remote_write | Standby received | Medium |
| on | Standby flushed | Lower |
| remote_apply | Standby applied | Lowest |

## Configuration

On the primary:

```sql
-- postgresql.conf
synchronous_commit = on
synchronous_standby_names = 'standby1'  -- Single sync standby

-- Multiple standbys with quorum
synchronous_standby_names = 'FIRST 2 (standby1, standby2, standby3)'

-- Any 2 of these must confirm
synchronous_standby_names = 'ANY 2 (standby1, standby2, standby3)'
```

Standbys identify themselves via `primary_conninfo`:

```sql
-- On standby postgresql.conf or connection string
primary_conninfo = 'host=primary application_name=standby1'
```

## Monitoring Synchronous Standbys

```sql
-- Check sync status
SELECT application_name, client_addr, sync_state, sync_priority
FROM pg_stat_replication;

-- sync_state values:
-- 'sync' = synchronous standby
-- 'async' = asynchronous standby
-- 'potential' = could become sync if current fails
```

## Per-Transaction Override

```sql
-- Use async for bulk loads
SET synchronous_commit = off;
INSERT INTO logs SELECT generate_series(1, 1000000);
SET synchronous_commit = on;
```

## Code Examples

```undefined
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