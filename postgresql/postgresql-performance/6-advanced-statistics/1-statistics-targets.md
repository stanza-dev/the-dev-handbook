---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-statistics-targets"
---

# Column Statistics

PostgreSQL collects statistics to estimate query costs.

## What Statistics Are Collected

For each column:
- **Null fraction**: % of NULL values
- **Distinct values**: Estimated unique values
- **Most common values (MCV)**: Frequent values and their frequencies
- **Histogram**: Distribution of non-MCV values

```sql
-- View column statistics
SELECT 
    attname AS column,
    n_distinct,
    null_frac,
    most_common_vals,
    most_common_freqs
FROM pg_stats
WHERE tablename = 'orders'
  AND attname = 'status';
```

## Statistics Target

Controls histogram bucket count (default: 100):

```sql
-- Increase for columns with skewed distribution
ALTER TABLE orders 
    ALTER COLUMN status SET STATISTICS 500;

-- Then refresh statistics
ANALYZE orders;
```

**When to increase:**
- Column has many distinct values
- Queries show bad row estimates
- Column has skewed distribution

**Trade-off:** Higher targets = better estimates, but slower ANALYZE

## Checking Estimate Accuracy

```sql
-- Compare estimated vs actual rows
EXPLAIN ANALYZE
SELECT * FROM orders WHERE status = 'pending';
```

Look for:
```
Seq Scan on orders  (cost=... rows=453 ...)
                    (actual ... rows=512 ...)
```

**Rules of thumb:**
- Estimates within 2x of actual: OK
- Estimates off by 10x+: Statistics problem

## Extended Statistics

For correlated columns:

```sql
-- Problem: planner doesn't know city and state are correlated
SELECT * FROM addresses 
WHERE city = 'San Francisco' AND state = 'CA';

-- Solution: Create extended statistics
CREATE STATISTICS addr_stats (dependencies, mcv)
ON city, state FROM addresses;

ANALYZE addresses;
```

### Types of Extended Statistics

| Type | Purpose |
|------|--------|
| `dependencies` | Functional dependencies between columns |
| `ndistinct` | Number of distinct combinations |
| `mcv` | Most common value combinations |

```sql
-- View extended statistics
SELECT stxname, stxkeys, stxkind
FROM pg_statistic_ext;
```

## Row Count Estimation Issues

```sql
-- If estimates are consistently wrong:

-- 1. Update statistics
ANALYZE problem_table;

-- 2. Increase statistics target
ALTER TABLE problem_table 
    ALTER COLUMN skewed_column SET STATISTICS 1000;
ANALYZE problem_table;

-- 3. Create extended statistics for correlated columns
CREATE STATISTICS corr_stats ON col1, col2 FROM problem_table;
ANALYZE problem_table;
```

📖 [Planner Statistics](https://www.postgresql.org/docs/18/planner-stats.html)

## Resources

- [Extended Statistics](https://www.postgresql.org/docs/18/planner-stats.html#PLANNER-STATS-EXTENDED) — Extended statistics for correlated columns

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*