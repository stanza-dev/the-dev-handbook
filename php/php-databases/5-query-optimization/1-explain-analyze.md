---
source_course: "php-databases"
source_lesson: "php-databases-explain-analyze"
---

# Using EXPLAIN to Analyze Queries

## Introduction

EXPLAIN reveals the database engine's execution plan for a query, showing which indexes it uses, how many rows it scans, and where bottlenecks hide. It is the single most important tool for diagnosing slow queries.

## Key Concepts

- **Execution Plan**: The sequence of operations the database will perform to execute a query.
- **Access Type**: How the database accesses each table (full scan, index scan, index lookup, etc.).
- **Key/Index**: Which index, if any, the optimizer chose for each table access.

## Real World Context

A query that works fine with 1,000 rows can grind to a halt with 1,000,000 rows. EXPLAIN catches these problems before users notice, showing you whether a full table scan is lurking behind a query that looks simple.

## Deep Dive

Basic EXPLAIN output in MySQL:

```php
<?php
$stmt = $pdo->query(
    'EXPLAIN SELECT * FROM users WHERE email = "john@example.com"'
);
print_r($stmt->fetch());

// Output:
// id: 1
// select_type: SIMPLE
// table: users
// type: const        <- Access type (const = unique index lookup)
// possible_keys: idx_email
// key: idx_email     <- Index actually used
// rows: 1            <- Estimated rows scanned
// Extra: NULL
```

Access types from best to worst:

| Type | Meaning | Action |
|------|---------|--------|
| system/const | Single row via unique index | Perfect |
| eq_ref | One row per join (unique key) | Excellent |
| ref | Multiple rows via non-unique index | Good |
| range | Index range scan (BETWEEN, <, >) | Acceptable |
| index | Full index scan (all index entries) | Review |
| ALL | Full table scan (every row) | Optimize! |

Common problems and their EXPLAIN signatures:

```php
<?php
// BAD: Function on column prevents index usage -> type: ALL
$pdo->query('EXPLAIN SELECT * FROM orders WHERE YEAR(created_at) = 2024');

// GOOD: Range condition uses index -> type: range
$pdo->query('EXPLAIN SELECT * FROM orders WHERE created_at >= "2024-01-01" AND created_at < "2025-01-01"');

// BAD: No index on status -> key: NULL
$pdo->query('EXPLAIN SELECT * FROM orders WHERE status = "pending"');
// After: CREATE INDEX idx_status ON orders(status);
// Now: key: idx_status
```

MySQL 8.0+ supports `EXPLAIN ANALYZE` which shows actual execution times:

```php
<?php
$stmt = $pdo->query('
    EXPLAIN ANALYZE
    SELECT u.name, COUNT(o.id) as order_count
    FROM users u
    LEFT JOIN orders o ON u.id = o.user_id
    GROUP BY u.id
');
// Shows actual time, rows processed, and loop iterations
```

The most common performance killers:

```sql
-- Problem: SELECT * fetches all columns
SELECT * FROM users;           -- Fetches unused columns
SELECT id, name FROM users;    -- Only what you need

-- Problem: Function on indexed column
WHERE LOWER(email) = 'john@example.com'  -- Breaks index
WHERE email = 'john@example.com'          -- Uses index

-- Problem: Leading wildcard in LIKE
WHERE name LIKE '%john%'  -- Full table scan
WHERE name LIKE 'john%'   -- Can use index
```

## Common Pitfalls

1. **Wrapping indexed columns in functions** — `WHERE YEAR(created_at) = 2024` cannot use an index on `created_at`. Rewrite as a range condition.
2. **Relying on row count estimates** — EXPLAIN shows estimated row counts. For actual counts, use `EXPLAIN ANALYZE` or check real query performance.

## Best Practices

1. **Run EXPLAIN on every query that touches large tables** — Make it a habit during development, not just when users complain.
2. **Look for type ALL and key NULL** — These two indicators immediately tell you a query is doing a full table scan without any index.

## Summary

- EXPLAIN shows the execution plan including access type, index usage, and estimated row count.
- Access types range from `const` (best) to `ALL` (full table scan, worst).
- Functions on indexed columns, leading wildcards in LIKE, and missing indexes are the most common causes of full scans.
- Use `EXPLAIN ANALYZE` in MySQL 8.0+ for actual execution times.

## Code Examples

**Query profiler that detects full table scans**

```php
<?php
declare(strict_types=1);

class QueryProfiler {
    public function __construct(private PDO $pdo) {}
    
    public function explain(string $query, array $params = []): array {
        $stmt = $this->pdo->prepare('EXPLAIN ' . $query);
        $stmt->execute($params);
        return $stmt->fetchAll();
    }
    
    public function hasFullTableScan(string $query, array $params = []): bool {
        $plan = $this->explain($query, $params);
        foreach ($plan as $step) {
            if ($step['type'] === 'ALL') {
                return true;
            }
        }
        return false;
    }
}

$profiler = new QueryProfiler($pdo);
if ($profiler->hasFullTableScan('SELECT * FROM orders WHERE status = ?', ['pending'])) {
    error_log('WARNING: Full table scan on orders query');
}
?>
```


## Resources

- [MySQL EXPLAIN](https://dev.mysql.com/doc/refman/8.0/en/explain.html) — MySQL EXPLAIN statement documentation

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*