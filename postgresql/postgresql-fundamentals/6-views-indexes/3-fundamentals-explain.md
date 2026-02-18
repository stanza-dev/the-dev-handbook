---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-explain"
---

# Query Analysis with EXPLAIN

EXPLAIN shows how PostgreSQL executes a query.

## Basic EXPLAIN

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

## EXPLAIN ANALYZE

**Actually runs the query** and shows real execution time:

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

## Reading the Output

- **Seq Scan**: Full table scan (often slow for large tables)
- **Index Scan**: Using an index (usually good)
- **cost**: Estimated cost (lower is better)
- **rows**: Estimated (and actual with ANALYZE) row count
- **width**: Average row size in bytes
- **actual time**: Real execution time in milliseconds

## Common Scan Types

| Scan Type | Description | Typical Performance |
|-----------|-------------|--------------------|
| Seq Scan | Full table scan | Slow for large tables |
| Index Scan | Uses index, fetches rows | Fast |
| Index Only Scan | Data from index only | Fastest |
| Bitmap Index Scan | Builds bitmap, then fetches | Good for many matches |

## Example: Before and After Index

```sql
-- Without index
EXPLAIN ANALYZE 
SELECT * FROM users WHERE email = 'alice@example.com';
-- Seq Scan: 50ms on 100,000 rows

-- Add index
CREATE INDEX idx_users_email ON users(email);

-- With index
EXPLAIN ANALYZE 
SELECT * FROM users WHERE email = 'alice@example.com';
-- Index Scan: 0.05ms
```

## EXPLAIN Options

```sql
-- More detailed output
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT * FROM books WHERE published_year > 2000;

-- JSON format (good for tools)
EXPLAIN (ANALYZE, FORMAT JSON)
SELECT * FROM books WHERE published_year > 2000;
```

## Red Flags to Watch For

1. **Seq Scan on large tables**: Add an index
2. **High row estimates vs actual**: Update statistics
3. **Nested loops with large tables**: Consider different join strategy
4. **Sort operations**: Add index with right order

📖 [EXPLAIN Documentation](https://www.postgresql.org/docs/18/sql-explain.html)

## Resources

- [Using EXPLAIN](https://www.postgresql.org/docs/18/using-explain.html) — How to use EXPLAIN for query optimization

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*