---
source_course: "postgresql-security"
source_lesson: "postgresql-security-pg-stat-statements"
---

# Monitoring with pg_stat_statements

## Introduction
The `pg_stat_statements` extension tracks execution statistics for all SQL statements. While primarily a performance tool, it is invaluable for security monitoring: detecting unusual query patterns, identifying unexpected table access, and building usage baselines for anomaly detection.

## Key Concepts
- **pg_stat_statements**: A built-in extension that records query statistics including call count, total time, rows returned, and shared buffer usage.
- **Query Normalization**: The extension normalizes queries by replacing literal values with parameters, grouping similar queries together.
- **Baseline Monitoring**: Establishing normal query patterns to detect anomalies that may indicate unauthorized access.

## Real World Context
After a suspected data breach, the security team needs to determine which tables were accessed and how frequently. pg_stat_statements shows that a role that normally runs 50 queries per hour suddenly executed 50,000 SELECTs against the `customers` table at 3 AM. This pattern immediately flags the incident for investigation.

## Deep Dive

### Installation

```ini
# postgresql.conf
shared_preload_libraries = 'pg_stat_statements'
pg_stat_statements.track = all
```

```sql
CREATE EXTENSION pg_stat_statements;
```

### Security-Relevant Queries

```sql
-- Top queries by execution count (detect unusual patterns)
SELECT userid::regrole, query, calls, total_exec_time,
       rows, mean_exec_time
FROM pg_stat_statements
ORDER BY calls DESC
LIMIT 20;

-- Queries accessing specific tables
SELECT userid::regrole, query, calls, rows
FROM pg_stat_statements
WHERE query ILIKE '%customers%'
ORDER BY calls DESC;

-- Unusual activity: queries with high row counts
SELECT userid::regrole, query, calls, rows,
       rows / NULLIF(calls, 0) AS avg_rows_per_call
FROM pg_stat_statements
WHERE rows / NULLIF(calls, 0) > 10000
ORDER BY rows DESC;
```

### Building Baselines

```sql
-- Snapshot statistics periodically
CREATE TABLE stat_snapshots (
    snapshot_time TIMESTAMPTZ DEFAULT NOW(),
    userid OID,
    query TEXT,
    calls BIGINT,
    total_exec_time DOUBLE PRECISION,
    rows BIGINT
);

INSERT INTO stat_snapshots (userid, query, calls, total_exec_time, rows)
SELECT userid, query, calls, total_exec_time, rows
FROM pg_stat_statements;

-- Reset counters after snapshot
SELECT pg_stat_statements_reset();
```

### Combining with pgAudit

Use pg_stat_statements for statistical monitoring and pgAudit for detailed logging. Together they provide both "what patterns are happening" and "exactly what happened" views of database activity.

## Common Pitfalls
1. **Not tracking enough queries** — The default `pg_stat_statements.max` is 5000. For large applications, increase this to avoid losing query statistics.
2. **Forgetting to periodically reset** — Without periodic resets, statistics accumulate from server start and long-term trends become hard to spot.

## Best Practices
1. **Snapshot and reset regularly** — Take hourly or daily snapshots of pg_stat_statements data, then reset counters to track trends over time.
2. **Alert on anomalous patterns** — Set up monitoring for unusual call counts, row counts, or new queries that were not seen before.

## Summary
- pg_stat_statements tracks execution statistics for all SQL, useful for both performance and security monitoring.
- Use it to detect unusual access patterns, unexpected table access, and establish baselines.
- Combine with pgAudit for comprehensive security observability.

## Code Examples

**Using pg_stat_statements to detect unusual access patterns on sensitive tables**

```sql
-- Detect unusual access patterns
SELECT userid::regrole AS role,
       left(query, 80) AS query_preview,
       calls, rows,
       rows / NULLIF(calls, 0) AS avg_rows
FROM pg_stat_statements
WHERE query ILIKE '%customers%'
ORDER BY calls DESC
LIMIT 10;
```


## Resources

- [pg_stat_statements](https://www.postgresql.org/docs/18/pgstatstatements.html) — Query statistics tracking extension

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*