---
source_course: "php-databases"
source_lesson: "php-databases-advanced-conditions"
---

# Advanced Query Conditions

## Introduction

Real-world queries need more than simple equality checks. IN clauses, NULL handling, OR groups, LIKE patterns, and subqueries are everyday requirements. Extending the query builder with these conditions while maintaining safety requires careful handling of edge cases.

## Key Concepts

- **IN Clause**: Filters rows where a column matches any value in a set.
- **Empty Array Edge Case**: An `IN ()` with no values is invalid SQL and must be handled explicitly.
- **Condition Groups**: Parenthesized groups of AND/OR conditions that control evaluation order.

## Real World Context

A product catalog filter might combine category IN (list), price range, search text, and optional availability. The query builder must compose these conditions safely without producing invalid SQL for empty filter lists.

## Deep Dive

The `whereIn` method must handle the empty array edge case. An empty `IN ()` is invalid SQL in every database:

```php
<?php
public function whereIn(string $column, array $values): self {
    if (empty($values)) {
        // Empty IN = impossible condition = no results
        return $this->whereRaw('1 = 0');
    }
    
    $clone = clone $this;
    $placeholders = [];
    
    foreach ($values as $i => $value) {
        $key = ':wherein_' . count($clone->wheres) . '_' . $i;
        $placeholders[] = $key;
        $clone->bindings[$key] = $value;
    }
    
    $clone->wheres[] = "$column IN (" . implode(', ', $placeholders) . ')';
    return $clone;
}
```

The `whereRaw('1 = 0')` trick ensures the query returns zero rows instead of crashing with a syntax error.

Similarly, `whereNotIn` with an empty array should return all rows (no exclusions):

```php
<?php
public function whereNotIn(string $column, array $values): self {
    if (empty($values)) {
        return $this; // Nothing to exclude
    }
    
    $clone = clone $this;
    $placeholders = [];
    foreach ($values as $i => $value) {
        $key = ':wherenotin_' . count($clone->wheres) . '_' . $i;
        $placeholders[] = $key;
        $clone->bindings[$key] = $value;
    }
    $clone->wheres[] = "$column NOT IN (" . implode(', ', $placeholders) . ')';
    return $clone;
}
```

NULL handling requires special SQL syntax since `= NULL` is always false:

```php
<?php
public function whereNull(string $column): self {
    $clone = clone $this;
    $clone->wheres[] = "$column IS NULL";
    return $clone;
}

public function whereNotNull(string $column): self {
    $clone = clone $this;
    $clone->wheres[] = "$column IS NOT NULL";
    return $clone;
}
```

Grouped conditions allow complex OR logic:

```php
<?php
public function whereGroup(callable $callback): self {
    $clone = clone $this;
    $group = new self($this->pdo);
    $callback($group);
    
    if ($group->wheres) {
        $clone->wheres[] = '(' . implode(' AND ', $group->wheres) . ')';
        $clone->bindings = array_merge($clone->bindings, $group->bindings);
    }
    return $clone;
}

// Usage
$results = $qb->table('users')
    ->where('status', 'active')
    ->whereGroup(function($q) {
        $q->where('role', 'admin');
        $q->orWhere('role', 'moderator');
    })
    ->get();
// WHERE status = :w0 AND (role = :w1 OR role = :w2)
```

LIKE queries for text search:

```php
<?php
public function whereLike(string $column, string $pattern): self {
    $clone = clone $this;
    $placeholder = ':like_' . count($clone->wheres);
    $clone->wheres[] = "$column LIKE $placeholder";
    $clone->bindings[$placeholder] = $pattern;
    return $clone;
}

$users = $qb->table('users')
    ->whereLike('email', '%@gmail.com')
    ->get();
```

## Common Pitfalls

1. **Not handling empty arrays in IN clauses** — Passing an empty array to `WHERE id IN ()` produces a SQL syntax error. Always check for empty and short-circuit.
2. **Using `= NULL` instead of `IS NULL`** — In SQL, `NULL = NULL` evaluates to NULL (falsy), not true. You must use `IS NULL` and `IS NOT NULL`.

## Best Practices

1. **Short-circuit empty IN clauses explicitly** — Use `1 = 0` for empty `IN` (no results) and skip the clause entirely for empty `NOT IN` (all results).
2. **Validate LIKE patterns** — If the pattern comes from user input, escape `%` and `_` characters unless you intentionally want wildcard behavior.

## Summary

- Handle the empty array edge case in IN clauses to prevent SQL syntax errors.
- Use `IS NULL` / `IS NOT NULL` instead of equality checks for NULL values.
- Grouped conditions with callbacks enable complex OR logic with correct parenthesization.
- Always escape or validate user-provided LIKE patterns.

## Code Examples

**Query builder with safe IN clause handling**

```php
<?php
declare(strict_types=1);

class QueryBuilder {
    private string $table = '';
    private array $columns = ['*'];
    private array $wheres = [];
    private array $bindings = [];
    private array $orderBy = [];
    private array $joins = [];
    private ?int $limit = null;
    private ?int $offset = null;
    
    public function __construct(private PDO $pdo) {}
    
    public function table(string $table): self {
        $clone = clone $this;
        $clone->table = $table;
        return $clone;
    }
    
    public function whereIn(string $column, array $values): self {
        if (empty($values)) {
            return $this->whereRaw('1 = 0');
        }
        $clone = clone $this;
        $keys = [];
        foreach ($values as $i => $value) {
            $key = ':in' . count($clone->bindings) . '_' . $i;
            $keys[] = $key;
            $clone->bindings[$key] = $value;
        }
        $clone->wheres[] = "$column IN (" . implode(', ', $keys) . ')';
        return $clone;
    }
    
    public function whereRaw(string $sql): self {
        $clone = clone $this;
        $clone->wheres[] = $sql;
        return $clone;
    }
}
?>
```


## Resources

- [SQL IN Operator](https://www.w3schools.com/sql/sql_in.asp) — SQL IN operator reference

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*