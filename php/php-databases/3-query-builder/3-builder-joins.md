---
source_course: "php-databases"
source_lesson: "php-databases-query-builder-joins"
---

# Joins & Aggregations

## Introduction

Joins combine rows from multiple tables, and aggregations compute summary values. A query builder that supports both allows complex reporting and data retrieval through the same fluent interface.

## Key Concepts

- **JOIN**: Combines rows from two tables based on a related column.
- **LEFT JOIN**: Returns all rows from the left table, with NULLs for non-matching right-table rows.
- **Aggregation**: Functions like COUNT, SUM, AVG, MIN, MAX that compute a single value from multiple rows.

## Real World Context

An admin dashboard showing "users with their order counts and total spend" requires a LEFT JOIN (to include users with zero orders) combined with GROUP BY and aggregate functions. A good query builder makes this readable.

## Deep Dive

Adding JOIN support to the query builder:

```php
<?php
class QueryBuilder {
    // ... existing properties ...
    private array $joins = [];
    private ?string $groupBy = null;
    private ?string $having = null;
    
    public function join(string $table, string $first, string $op, string $second): self {
        $clone = clone $this;
        $clone->joins[] = "JOIN $table ON $first $op $second";
        return $clone;
    }
    
    public function leftJoin(string $table, string $first, string $op, string $second): self {
        $clone = clone $this;
        $clone->joins[] = "LEFT JOIN $table ON $first $op $second";
        return $clone;
    }
    
    public function groupBy(string ...$columns): self {
        $clone = clone $this;
        $clone->groupBy = implode(', ', $columns);
        return $clone;
    }
    
    public function having(string $condition, mixed $value): self {
        $clone = clone $this;
        $placeholder = ':having_0';
        $clone->having = "$condition $placeholder";
        $clone->bindings[$placeholder] = $value;
        return $clone;
    }
    
    public function count(): int {
        $clone = clone $this;
        $clone->columns = ['COUNT(*) as cnt'];
        $clone->orderBy = [];
        $clone->limit = null;
        $result = $clone->first();
        return (int) ($result['cnt'] ?? 0);
    }
    
    public function sum(string $column): float {
        $clone = clone $this;
        $clone->columns = ["SUM($column) as total"];
        $clone->orderBy = [];
        $result = $clone->first();
        return (float) ($result['total'] ?? 0);
    }
}
```

Using joins with aggregations:

```php
<?php
$qb = new QueryBuilder($pdo);

// Users with order statistics
$stats = $qb
    ->table('users u')
    ->select('u.id', 'u.name', 'COUNT(o.id) as order_count', 'COALESCE(SUM(o.total), 0) as total_spent')
    ->leftJoin('orders o', 'u.id', '=', 'o.user_id')
    ->where('u.status', 'active')
    ->groupBy('u.id', 'u.name')
    ->orderBy('total_spent', 'DESC')
    ->limit(10)
    ->get();
```

The generated SQL is a single efficient query that would otherwise be an N+1 problem.

Self-joins for hierarchical data:

```php
<?php
$categories = $qb
    ->table('categories c')
    ->select('c.id', 'c.name', 'p.name as parent_name')
    ->leftJoin('categories p', 'c.parent_id', '=', 'p.id')
    ->orderBy('c.name')
    ->get();
```

With PHP 8.5's pipe operator, post-processing aggregation results becomes fluid:

```php
<?php
$topSpenders = $qb
    ->table('users u')
    ->select('u.name', 'SUM(o.total) as spent')
    ->join('orders o', 'u.id', '=', 'o.user_id')
    ->groupBy('u.id', 'u.name')
    ->orderBy('spent', 'DESC')
    ->limit(5)
    ->get()
    |> (fn($rows) => array_column($rows, 'name'));
// Returns just the names of top 5 spenders
```

## Common Pitfalls

1. **Using JOIN instead of LEFT JOIN for optional relationships** — A regular JOIN excludes parent rows that have no matching child rows, silently dropping data from results.
2. **Forgetting GROUP BY with aggregations** — Selecting non-aggregated columns alongside COUNT/SUM without GROUP BY produces undefined behavior in strict SQL mode.

## Best Practices

1. **Always use LEFT JOIN for optional relationships** — Use INNER JOIN only when you specifically want to exclude unmatched rows.
2. **Use COALESCE with SUM** — `SUM()` returns NULL when no rows match. `COALESCE(SUM(col), 0)` returns zero instead, preventing null propagation.

## Summary

- JOIN and LEFT JOIN combine data from multiple tables in a single query.
- Aggregation functions (COUNT, SUM, AVG) compute summary statistics with GROUP BY.
- Use LEFT JOIN for optional relationships and COALESCE to handle NULL aggregates.
- PHP 8.5's pipe operator streamlines post-query result transformations.

## Code Examples

**Join with aggregation for dashboard stats**

```php
<?php
declare(strict_types=1);

$qb = new QueryBuilder($pdo);

// Dashboard: users with order stats
$dashboard = $qb
    ->table('users u')
    ->select(
        'u.id',
        'u.name',
        'u.email',
        'COUNT(o.id) as order_count',
        'COALESCE(SUM(o.total), 0) as total_spent',
        'MAX(o.created_at) as last_order'
    )
    ->leftJoin('orders o', 'u.id', '=', 'o.user_id')
    ->where('u.status', 'active')
    ->groupBy('u.id', 'u.name', 'u.email')
    ->orderBy('total_spent', 'DESC')
    ->limit(20)
    ->get();
?>
```


## Resources

- [MySQL JOIN Syntax](https://dev.mysql.com/doc/refman/8.0/en/join.html) — MySQL JOIN types and syntax

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*