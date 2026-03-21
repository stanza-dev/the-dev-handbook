---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-statistics-targets"
---

# Column Statistics

## Introduction

PostgreSQL collects detailed statistics about each column to estimate how many rows a query will return. These estimates drive the planner's cost calculations and ultimately determine which execution plan is chosen. Inaccurate statistics lead to poor plans, while properly configured statistics produce optimal query performance.

## Key Concepts

- **n_distinct**: The estimated number of distinct values in a column. Negative values indicate a ratio to table size.
- **Most Common Values (MCV)**: The most frequent values and their probabilities, used for equality comparisons.
- **Histogram**: A distribution of non-MCV values divided into equal-frequency buckets.
- **Statistics target**: The number of histogram buckets and MCV entries collected (default: 100).
- **Extended statistics**: Multi-column statistics that capture correlations between columns.

## Real World Context

A query filtering on `city = 'San Francisco' AND state = 'CA'` should return very few rows, but without extended statistics, PostgreSQL treats the two conditions as independent and may underestimate selectivity by orders of magnitude. This can lead to choosing a nested loop join when a hash join would be faster, turning a 10ms query into a 10-second one.

## Deep Dive

### What Statistics Are Collected

For each column, PostgreSQL tracks null fraction, distinct values, most common values and their frequencies, and a histogram of value distribution:

```sql
SELECT 
    attname AS column_name,
    n_distinct,
    null_frac,
    most_common_vals,
    most_common_freqs
FROM pg_stats
WHERE tablename = 'orders'
  AND attname = 'status';
```

This shows the data distribution PostgreSQL uses for query planning.

### Statistics Target

The statistics target controls histogram bucket count. The default is 100, which is sufficient for most columns:

```sql
-- Increase for columns with skewed distribution
ALTER TABLE orders 
    ALTER COLUMN status SET STATISTICS 500;

-- Then refresh statistics
ANALYZE orders;
```

Increase the target when columns have many distinct values, queries show bad row estimates, or the column has a heavily skewed distribution. The trade-off is that higher targets produce better estimates but make ANALYZE slower.

### Checking Estimate Accuracy

Compare estimated vs actual rows to detect statistics problems:

```sql
EXPLAIN ANALYZE
SELECT * FROM orders WHERE status = 'pending';
```

Look at the difference between estimated and actual rows:

```
Seq Scan on orders  (cost=... rows=453 ...)
                    (actual ... rows=512 ...)
```

Estimates within 2x of actual are fine. Estimates off by 10x or more indicate a statistics problem that needs attention.

### Extended Statistics

Standard statistics treat columns independently. For correlated columns, you need extended statistics:

```sql
-- Problem: planner doesn't know city and state are correlated
SELECT * FROM addresses 
WHERE city = 'San Francisco' AND state = 'CA';

-- Solution: Create extended statistics
CREATE STATISTICS addr_stats (dependencies, mcv)
ON city, state FROM addresses;

ANALYZE addresses;
```

Extended statistics come in three types:

| Type | Purpose |
|------|--------|
| `dependencies` | Functional dependencies between columns |
| `ndistinct` | Number of distinct combinations |
| `mcv` | Most common value combinations |

You can inspect them:

```sql
SELECT stxname, stxkeys, stxkind
FROM pg_statistic_ext;
```

This shows which extended statistics exist and what types they track.

### Fixing Row Count Estimation Issues

A systematic approach to fixing bad estimates:

```sql
-- 1. Update statistics
ANALYZE problem_table;

-- 2. Increase statistics target for skewed columns
ALTER TABLE problem_table 
    ALTER COLUMN skewed_column SET STATISTICS 1000;
ANALYZE problem_table;

-- 3. Create extended statistics for correlated columns
CREATE STATISTICS corr_stats ON col1, col2 FROM problem_table;
ANALYZE problem_table;
```

Always run ANALYZE after changing statistics targets or creating extended statistics.

## Common Pitfalls

1. **Ignoring bad row estimates** - When EXPLAIN ANALYZE shows estimated rows off by 10x or more, the planner is likely choosing a suboptimal plan. Investigate and fix the statistics.
2. **Setting statistics target too high globally** - `ALTER SYSTEM SET default_statistics_target = 1000` slows down ANALYZE on every column. Only increase it for specific columns that need it.
3. **Not creating extended statistics for correlated filters** - Any query that filters on two or more correlated columns (country+city, year+month, product_type+brand) benefits from extended statistics.

## Best Practices

1. **Check estimated vs actual rows** - Make it a habit to run EXPLAIN ANALYZE on important queries and verify that row estimates are within 2-3x of actual.
2. **Use extended statistics for correlated columns** - Create them proactively for columns that are commonly filtered together.
3. **Increase statistics target selectively** - Only raise it for columns where the default 100 buckets are not enough to represent the distribution.

## Summary

- PostgreSQL collects per-column statistics including distinct values, MCVs, and histograms.
- The statistics target controls histogram resolution; increase it for skewed or high-cardinality columns.
- Bad row estimates (10x+ off) indicate statistics problems that lead to poor plan choices.
- Extended statistics capture correlations between columns that single-column stats miss.
- Always run ANALYZE after changing statistics targets or creating extended statistics.

## Code Examples

**Inspecting column statistics, creating extended statistics for correlated columns, and increasing statistics targets**

```sql
-- Check column statistics for a specific column
SELECT attname, n_distinct, null_frac,
       most_common_vals, most_common_freqs
FROM pg_stats
WHERE tablename = 'orders' AND attname = 'status';

-- Create extended statistics for correlated columns
CREATE STATISTICS addr_stats (dependencies, mcv)
ON city, state FROM addresses;
ANALYZE addresses;

-- Increase statistics target for skewed column
ALTER TABLE orders ALTER COLUMN status SET STATISTICS 500;
ANALYZE orders;
```


## Resources

- [Planner Statistics](https://www.postgresql.org/docs/18/planner-stats.html) — How PostgreSQL collects and uses planner statistics
- [Extended Statistics](https://www.postgresql.org/docs/18/planner-stats.html#PLANNER-STATS-EXTENDED) — Extended statistics for correlated columns

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*