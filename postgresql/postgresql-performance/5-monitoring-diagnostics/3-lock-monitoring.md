---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-lock-monitoring"
---

# Lock Monitoring and Wait Events

## Introduction

Lock conflicts are among the most difficult performance problems to diagnose. A query may be fast in isolation but wait minutes when another transaction holds a conflicting lock. PostgreSQL provides `pg_locks`, wait event tracking, and advisory locks to help you understand and resolve contention issues.

## Key Concepts

- **pg_locks**: A system view showing all current locks held and awaited by active transactions.
- **Wait events**: Indicators of what a backend is currently waiting for, visible in `pg_stat_activity.wait_event_type` and `wait_event`.
- **Deadlock**: A situation where two or more transactions each hold locks the other needs, forming a cycle that PostgreSQL must break by aborting one.
- **Advisory locks**: Application-level locks that PostgreSQL manages but does not automatically acquire or release.
- **pg_stat_io**: A statistics view tracking I/O operations by backend type and context.

## Real World Context

A deployment runs a schema migration that takes an ACCESS EXCLUSIVE lock on a table. All queries against that table queue up, and within seconds your application has hundreds of waiting connections. Understanding lock types, how to monitor them, and how to design migrations that avoid exclusive locks prevents these cascading failures.

## Deep Dive

### Understanding pg_locks

The `pg_locks` view shows all locks currently held or awaited:

```sql
-- View current locks and identify conflicts
SELECT 
    l.pid,
    l.locktype,
    l.mode,
    l.granted,
    l.relation::regclass AS table_name,
    a.state,
    LEFT(a.query, 60) AS query
FROM pg_locks l
JOIN pg_stat_activity a ON l.pid = a.pid
WHERE l.relation IS NOT NULL
  AND NOT l.granted
ORDER BY l.pid;
```

Rows with `granted = false` are waiting for a lock. Joining with `pg_stat_activity` shows what query is blocked and what it is waiting on.

### Identifying Lock Conflicts

To find which sessions are blocking others:

```sql
-- Find blocking and blocked sessions
SELECT 
    blocked.pid AS blocked_pid,
    LEFT(blocked_activity.query, 50) AS blocked_query,
    blocking.pid AS blocking_pid,
    LEFT(blocking_activity.query, 50) AS blocking_query,
    blocked.mode AS blocked_mode
FROM pg_locks blocked
JOIN pg_locks blocking 
    ON blocking.locktype = blocked.locktype
    AND blocking.relation = blocked.relation
    AND blocking.pid != blocked.pid
    AND blocking.granted = true
JOIN pg_stat_activity blocked_activity ON blocked.pid = blocked_activity.pid
JOIN pg_stat_activity blocking_activity ON blocking.pid = blocking_activity.pid
WHERE NOT blocked.granted;
```

This reveals the complete blocking chain, showing which query is holding the lock and which is waiting.

### Wait Events

The `wait_event_type` and `wait_event` columns in `pg_stat_activity` show what each backend is currently waiting for:

```sql
-- Check what backends are waiting on
SELECT 
    pid,
    wait_event_type,
    wait_event,
    state,
    LEFT(query, 60) AS query
FROM pg_stat_activity
WHERE wait_event IS NOT NULL
  AND state = 'active';
```

Common wait event types include Lock (waiting for a heavyweight lock), LWLock (lightweight internal locks), IO (waiting for disk I/O), and BufferPin (waiting for a buffer pin).

### Identifying and Resolving Deadlocks

Deadlocks occur when transactions form a circular wait. PostgreSQL detects them automatically and aborts one transaction:

```sql
-- PostgreSQL logs deadlock details. Check the log for:
-- ERROR: deadlock detected
-- DETAIL: Process 1234 waits for ShareLock on transaction 5678;
--         blocked by process 9012.
--         Process 9012 waits for ShareLock on transaction 1234;
--         blocked by process 1234.

-- To reduce deadlock frequency, access tables in consistent order:
-- Always lock table_a before table_b in all transactions
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
SELECT * FROM accounts WHERE id = 2 FOR UPDATE;
-- ... perform operations ...
COMMIT;
```

Consistent lock ordering is the most effective deadlock prevention strategy.

### Advisory Locks

Advisory locks are application-level locks managed by PostgreSQL:

```sql
-- Session-level advisory lock (held until session ends or explicit unlock)
SELECT pg_advisory_lock(12345);
-- ... do exclusive work ...
SELECT pg_advisory_unlock(12345);

-- Transaction-level advisory lock (released at COMMIT/ROLLBACK)
SELECT pg_advisory_xact_lock(12345);

-- Try to acquire without blocking
SELECT pg_try_advisory_lock(12345);
-- Returns true if acquired, false if already held
```

Advisory locks are useful for application-level mutual exclusion (e.g., ensuring only one worker processes a specific job).

### PG18: pg_stat_io Enhancements

PostgreSQL 18 adds byte-level columns to `pg_stat_io` for more accurate I/O analysis:

```sql
-- PG18: I/O statistics with byte-level detail
SELECT 
    backend_type,
    io_object,
    io_context,
    reads,
    pg_size_pretty(read_bytes) AS read_volume,
    writes,
    pg_size_pretty(write_bytes) AS write_volume,
    extends,
    pg_size_pretty(extend_bytes) AS extend_volume
FROM pg_stat_io
WHERE reads > 0 OR writes > 0
ORDER BY read_bytes DESC NULLS LAST;
```

The `read_bytes`, `write_bytes`, and `extend_bytes` columns provide actual data volumes rather than just page counts, making it easier to understand I/O workload distribution.

## Common Pitfalls

1. **Ignoring lock waits in monitoring** - Slow queries are not always CPU or I/O bound. They may simply be waiting for a lock. Always check `wait_event_type` in `pg_stat_activity`.
2. **Running DDL during peak hours** - ALTER TABLE, CREATE INDEX (without CONCURRENTLY), and other DDL statements take locks that can block all queries. Schedule them during low-traffic periods.
3. **Forgetting to release advisory locks** - Session-level advisory locks persist until the session ends or explicit unlock. Use transaction-level locks when possible to ensure automatic release.

## Best Practices

1. **Use CREATE INDEX CONCURRENTLY** - Building indexes concurrently avoids blocking writes, at the cost of taking longer.
2. **Set lock_timeout for migrations** - Use `SET lock_timeout = '5s'` before DDL to fail fast if a lock cannot be acquired, preventing connection pile-ups.
3. **Monitor wait events continuously** - Include `wait_event_type` distribution in your monitoring dashboards to catch lock contention early.

## Summary

- `pg_locks` reveals all current locks and shows which sessions are blocked.
- Wait events in `pg_stat_activity` indicate what backends are waiting on (locks, I/O, buffer pins).
- Deadlocks are detected automatically by PostgreSQL; prevent them with consistent lock ordering.
- Advisory locks provide application-level mutual exclusion managed by PostgreSQL.
- PG18 enhances `pg_stat_io` with byte-level I/O tracking for accurate workload analysis.

## Code Examples

**Identifying lock conflicts and wait events to diagnose blocking and contention issues**

```sql
-- Find blocked and blocking sessions
SELECT 
    blocked.pid AS blocked_pid,
    LEFT(blocked_act.query, 50) AS blocked_query,
    blocking.pid AS blocking_pid,
    LEFT(blocking_act.query, 50) AS blocking_query
FROM pg_locks blocked
JOIN pg_locks blocking 
    ON blocking.locktype = blocked.locktype
    AND blocking.relation = blocked.relation
    AND blocking.pid != blocked.pid
    AND blocking.granted
JOIN pg_stat_activity blocked_act ON blocked.pid = blocked_act.pid
JOIN pg_stat_activity blocking_act ON blocking.pid = blocking_act.pid
WHERE NOT blocked.granted;

-- Check wait events
SELECT pid, wait_event_type, wait_event, LEFT(query, 60)
FROM pg_stat_activity
WHERE wait_event IS NOT NULL AND state = 'active';
```


## Resources

- [Monitoring Locks](https://www.postgresql.org/docs/18/view-pg-locks.html) — pg_locks view documentation
- [Wait Events](https://www.postgresql.org/docs/18/monitoring-stats.html#WAIT-EVENT-TABLE) — Wait event types and their meanings

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*