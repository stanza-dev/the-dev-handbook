---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-explain"
---

# Query Analysis with EXPLAIN

## Introduction
EXPLAIN shows how PostgreSQL plans to execute a query — which tables to scan, which indexes to use, and in what order. EXPLAIN ANALYZE goes further by actually running the query and showing real execution times. These tools are essential for diagnosing slow queries.

## Key Concepts
- **EXPLAIN**: Shows the query execution plan without running the query. Displays estimated costs and row counts.
- **EXPLAIN ANALYZE**: Runs the query and shows actual execution times alongside estimates.
- **Seq Scan**: A full table scan — reading every row. Slow for large tables when you need only a few rows.
- **Index Scan**: Using an index to find rows directly. Much faster for selective queries.
- **Cost**: An arbitrary unit measuring the estimated expense of an operation. Lower is better.

## Real World Context
Every production database issue starts with EXPLAIN ANALYZE. When an API endpoint is slow, you run EXPLAIN on its queries to find the bottleneck — usually a missing index or a bad join strategy. Understanding EXPLAIN output is the most practical performance skill a developer can have.

## Deep Dive

### Basic EXPLAIN

EXPLAIN shows the query plan without running it:

```sql
EXPLAIN SELECT * FROM books WHERE published_year > 2000;
```

Output:
```
                        QUERY PLAN
----------------------------------------------------------
 Seq Scan on books  (cost=0.00..1.15 rows=5 width=48)
   Filter: (published_year > 2000)
```

This tells you PostgreSQL will do a sequential scan (read every row) and filter by published_year.

### EXPLAIN ANALYZE

EXPLAIN ANALYZE actually runs the query and shows real times:

```sql
EXPLAIN ANALYZE SELECT * FROM books WHERE published_year > 2000;
```

Output:
```
                                           QUERY PLAN
-------------------------------------------------------------------------------------------------
 Seq Scan on books  (cost=0.00..1.15 rows=5 width=48) (actual time=0.015..0.018 rows=4 loops=1)
   Filter: (published_year > 2000)
   Rows Removed by Filter: 6
 Planning Time: 0.065 ms
 Execution Time: 0.033 ms
```

The "actual" numbers show real performance. Notice "rows=5" (estimated) vs "rows=4" (actual) — close estimates indicate healthy statistics.

### Reading the Output

Key fields in EXPLAIN output:

- **Seq Scan**: Full table scan (often slow for large tables)
- **Index Scan**: Using an index (usually fast)
- **cost**: Estimated cost (lower is better)
- **rows**: Estimated (and actual with ANALYZE) row count
- **width**: Average row size in bytes
- **actual time**: Real execution time in milliseconds

### Common Scan Types

| Scan Type | Description | Performance |
|-----------|-------------|--------------------|
| Seq Scan | Full table scan | Slow for large tables |
| Index Scan | Uses index, fetches rows | Fast |
| Index Only Scan | Data from index only | Fastest |
| Bitmap Index Scan | Builds bitmap, then fetches | Good for many matches |

Index Only Scan is the fastest because it reads data directly from the index without touching the table.

### Before and After Index

EXPLAIN shows the dramatic impact of adding an index:

```sql
-- Without index: Seq Scan
EXPLAIN ANALYZE 
SELECT * FROM users WHERE email = 'alice@example.com';
-- Seq Scan: 50ms on 100,000 rows

-- Add index
CREATE INDEX idx_users_email ON users(email);

-- With index: Index Scan
EXPLAIN ANALYZE 
SELECT * FROM users WHERE email = 'alice@example.com';
-- Index Scan: 0.05ms
```

The query went from 50ms to 0.05ms — a 1000x improvement.

### EXPLAIN Options

EXPLAIN supports several output formats:

```sql
-- More detailed output with buffer information
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT * FROM books WHERE published_year > 2000;

-- JSON format (useful for visualization tools)
EXPLAIN (ANALYZE, FORMAT JSON)
SELECT * FROM books WHERE published_year > 2000;
```

BUFFERS shows how many disk pages were read, which helps identify I/O bottlenecks.

### Red Flags to Watch For

1. **Seq Scan on large tables**: Usually means a missing index
2. **High estimated vs actual rows**: Statistics may be outdated — run ANALYZE
3. **Nested loops with large tables**: May need a different join strategy
4. **Sort operations**: Consider adding an index with the right column order

## Common Pitfalls
1. **Only using EXPLAIN without ANALYZE** — EXPLAIN shows estimates, not reality. Always use EXPLAIN ANALYZE to see actual performance (but be careful with UPDATE/DELETE — they actually run!).
2. **Ignoring the Planning Time** — If Planning Time is high relative to Execution Time, the query may have too many JOINs or complex expressions.

## Best Practices
1. **Run EXPLAIN ANALYZE on slow queries first** — Before adding indexes blindly, understand what PostgreSQL is actually doing.
2. **Compare estimated vs actual rows** — Large differences indicate outdated statistics. Run `ANALYZE tablename` to update them.
3. **Use EXPLAIN (ANALYZE, BUFFERS)** — The BUFFERS option shows I/O activity, which is often the real bottleneck.

## Summary
- EXPLAIN shows the query plan; EXPLAIN ANALYZE runs the query and shows actual times.
- Seq Scan reads every row; Index Scan uses an index for fast lookups.
- Cost is an estimated unit; actual time is the real execution time.
- Compare estimated vs actual rows to check if statistics are current.
- Red flags include Seq Scan on large tables, high row estimate mismatches, and sort operations.

## Code Examples

**Step-by-step query optimization: diagnose with EXPLAIN ANALYZE, add an index, then verify the improvement**

```sql
-- Diagnose a slow query step by step
-- 1. Check the current plan
EXPLAIN ANALYZE
SELECT u.name, COUNT(o.id) AS order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.name
ORDER BY order_count DESC;

-- 2. If Seq Scan on orders, add an index
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- 3. Re-run EXPLAIN ANALYZE to verify improvement
```


## Resources

- [Using EXPLAIN](https://www.postgresql.org/docs/18/using-explain.html) — How to use EXPLAIN for query optimization

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*