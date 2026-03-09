---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-query-analysis"
---

# Query Analysis and EXPLAIN

## Introduction
Understanding how your database executes queries is the key to optimization. The EXPLAIN command reveals whether queries use indexes, how many rows are scanned, and where bottlenecks occur.

## Key Concepts
- **EXPLAIN**: A database command that shows the query execution plan without running the query.
- **Seq Scan**: A full table scan — reads every row. Usually indicates a missing index.
- **Index Scan**: Uses an index to find rows quickly. This is what you want.

## Real World Context
A slow admin dashboard query might scan 2 million rows when it only needs 50 results. Running EXPLAIN reveals a missing index — adding it drops the query from 8 seconds to 2 milliseconds.

## Deep Dive

### Using EXPLAIN in Rails

```ruby
# Basic explain
Post.where(user_id: 1).explain

# With analysis (actually runs the query)
Post.where(user_id: 1).explain(:analyze)
```

### Reading EXPLAIN Output (PostgreSQL)

```
Seq Scan on posts (cost=0.00..25000.00 rows=500 width=100)
  Filter: (status = 'published')
```

Key terms:
- **Seq Scan**: Full table scan (usually bad for large tables)
- **Index Scan**: Using an index (good)
- **cost**: Estimated cost (lower is better)
- **rows**: Estimated rows returned

### Good vs Bad Plans

```sql
-- BAD: Sequential scan on large table
Seq Scan on posts (cost=0.00..25000.00 rows=500)
  Filter: (status = 'published')

-- GOOD: Index scan
Index Scan using index_posts_on_status on posts (cost=0.00..50.00 rows=500)
  Index Cond: (status = 'published')
```

### Functions Prevent Index Usage

```ruby
# BAD: function on indexed column prevents index use
User.where('LOWER(email) = ?', email.downcase)

# GOOD: normalize data or use expression index
User.where(email: email.downcase)
# Or create expression index:
# add_index :users, 'LOWER(email)'
```

### Slow Query Logging

```ruby
# config/environments/production.rb
config.active_record.warn_on_records_fetched_greater_than = 1000
```

## Common Pitfalls
1. **Ignoring Seq Scans on large tables** — A sequential scan on a 10-row table is fine. On a million-row table, it's a critical problem.
2. **Using functions on indexed columns** — `WHERE LOWER(email) = ?` bypasses the index. Use expression indexes or normalize data.

## Best Practices
1. **Run EXPLAIN on slow queries** — Any query over 100ms deserves an EXPLAIN analysis.
2. **Use pg_stat_statements in production** — This PostgreSQL extension tracks query performance over time, revealing which queries consume the most resources.

## Summary
- EXPLAIN shows whether queries use indexes or do full table scans.
- Seq Scan on large tables usually means a missing index.
- Functions on indexed columns prevent index usage.
- Use `explain(:analyze)` to see actual execution times.

## Code Examples

**Using EXPLAIN to identify a missing index — Seq Scan means the database is reading every row**

```ruby
# Check if a query uses an index
puts Post.where(user_id: 1).explain
# => Index Scan using index_posts_on_user_id...

# Get actual execution time
puts Post.where(status: 'published').explain(:analyze)
# => Seq Scan on posts (actual time=0.015..125.432)
# Seq Scan = missing index! Add one:
# add_index :posts, :status
```


## Resources

- [PostgreSQL EXPLAIN Documentation](https://www.postgresql.org/docs/current/sql-explain.html) — PostgreSQL official documentation on the EXPLAIN command

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*