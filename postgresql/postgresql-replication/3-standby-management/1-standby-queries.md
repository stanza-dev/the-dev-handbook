---
source_course: "postgresql-replication"
source_lesson: "postgresql-replication-hot-standby-queries"
---

# Running Queries on Hot Standby

## Introduction

Hot Standby allows you to run read-only queries on a standby server while it continues to receive and replay WAL. This enables read scaling and reporting without impacting the primary server.

This lesson covers how to configure hot standby, manage query conflicts with WAL replay, and monitor conflict statistics.

## Key Concepts

- **Hot Standby**: A standby server mode that accepts read-only connections while replaying WAL from the primary.
- **Query Conflict**: A situation where WAL replay needs to modify data that a standby query is currently reading, forcing one to wait or be canceled.
- **hot_standby_feedback**: When enabled, the standby informs the primary about its oldest running query, preventing the primary from vacuuming rows the standby needs.
- **max_standby_streaming_delay**: The maximum time WAL replay will wait before canceling conflicting queries on the standby.

## Real World Context

An analytics team runs long-running reports against a hot standby to avoid impacting the primary. However, when the primary vacuums old rows, WAL replay on the standby must remove those same rows, which conflicts with the running report query. Without proper configuration, the report is canceled mid-execution. Setting `hot_standby_feedback = on` prevents this by telling the primary not to vacuum rows the standby is still reading.

## Deep Dive

Enable hot standby on the standby server:

```sql
-- On standby server (postgresql.conf)
hot_standby = on
```

This single setting enables read-only connections to the standby.

Standby queries have restrictions compared to a primary:

```sql
-- These work on hot standby
SELECT * FROM users;
EXPLAIN ANALYZE SELECT * FROM orders;

-- These fail on hot standby
CREATE TABLE test (id int);  -- ERROR: cannot execute in read-only transaction
INSERT INTO users (name) VALUES ('test');  -- ERROR
```

Only read-only operations are permitted; any write attempt returns an error.

WAL replay can conflict with running queries. Configure conflict handling:

```sql
-- Configure conflict handling
max_standby_streaming_delay = 30s   -- Wait before canceling queries
max_standby_archive_delay = 30s
hot_standby_feedback = on           -- Tell primary about running queries
```

With `hot_standby_feedback = on`, the primary will not vacuum rows the standby needs. This prevents most conflicts but can cause table bloat on the primary if the standby runs very long queries.

Monitor conflicts using the database statistics view:

```sql
-- Check for replay conflicts
SELECT datname, confl_tablespace, confl_lock, confl_snapshot,
       confl_bufferpin, confl_deadlock
FROM pg_stat_database_conflicts;
```

High values in `confl_snapshot` indicate that vacuum-related conflicts are frequent and you may need to increase `max_standby_streaming_delay` or enable `hot_standby_feedback`.

## Common Pitfalls

- **Setting max_standby_streaming_delay = -1 without caution**: This prevents all query cancellations but allows the standby to fall arbitrarily far behind the primary, which defeats the purpose of near-real-time replication.
- **Enabling hot_standby_feedback without monitoring primary bloat**: Long standby queries can prevent vacuum from cleaning up dead rows on the primary, causing table bloat.
- **Running writes on a standby**: Any write operation fails immediately; applications must handle this gracefully with read-only connection strings.

## Best Practices

- Enable `hot_standby_feedback` for standbys running analytical queries to reduce conflicts.
- Set `max_standby_streaming_delay` to a value that balances query completion against replication lag.
- Monitor `pg_stat_database_conflicts` and adjust settings based on actual conflict frequency.

## Summary

- Hot Standby enables read-only queries on a standby server during WAL replay.
- WAL replay can conflict with running queries; configure `max_standby_streaming_delay` to control cancellation behavior.
- `hot_standby_feedback` tells the primary to preserve rows needed by standby queries, reducing conflicts at the cost of potential primary bloat.
- Monitor conflicts via `pg_stat_database_conflicts` and tune settings accordingly.

## Code Examples

**Hot Standby Configuration**

```sql
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