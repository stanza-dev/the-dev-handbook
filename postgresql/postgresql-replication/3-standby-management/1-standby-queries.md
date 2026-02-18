---
source_course: "postgresql-replication"
source_lesson: "hot-standby-queries"
---

# Running Queries on Hot Standby

Hot Standby allows you to run read-only queries on a standby server while it continues to receive and replay WAL. This enables read scaling and reporting without impacting the primary.

## Enabling Hot Standby

```sql
-- On standby server (postgresql.conf)
hot_standby = on
```

## Hot Standby Limitations

Standby queries have restrictions:

```sql
-- These work on hot standby
SELECT * FROM users;
EXPLAIN ANALYZE SELECT * FROM orders;

-- These fail on hot standby
CREATE TABLE test (id int);  -- ERROR: cannot execute in read-only transaction
INSERT INTO users (name) VALUES ('test');  -- ERROR
```

## Query Conflicts

WAL replay can conflict with running queries:

```sql
-- Configure conflict handling
max_standby_streaming_delay = 30s   -- Wait before canceling queries
max_standby_archive_delay = 30s
hot_standby_feedback = on           -- Tell primary about running queries
```

With `hot_standby_feedback = on`, the primary won't vacuum rows the standby needs.

## Monitoring Conflicts

```sql
-- Check for replay conflicts
SELECT datname, confl_tablespace, confl_lock, confl_snapshot,
       confl_bufferpin, confl_deadlock
FROM pg_stat_database_conflicts;
```

## Read-Only Connection Enforcement

```sql
-- Force all connections to be read-only
default_transaction_read_only = on

-- Or per-session
SET transaction_read_only = on;
```

## Code Examples

```undefined
-- Standby postgresql.conf
hot_standby = on
hot_standby_feedback = on
max_standby_streaming_delay = 60s
max_standby_archive_delay = 60s
```


## Resources

- [Hot Standby](https://www.postgresql.org/docs/current/hot-standby.html) — undefined

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*