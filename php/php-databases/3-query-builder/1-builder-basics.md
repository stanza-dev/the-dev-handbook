---
source_course: "php-databases"
source_lesson: "php-databases-query-builder-basics"
---

# Building a Query Builder

## Introduction

Query builders provide a fluent, object-oriented interface for constructing SQL queries safely. Instead of concatenating SQL strings, you chain method calls that generate parameterized queries, eliminating SQL injection by design.

## Key Concepts

- **Fluent Interface**: A design pattern where methods return `$this` (or a clone) so calls can be chained.
- **Immutable Builder**: Each method returns a new instance rather than mutating the current one, preventing accidental side effects.
- **Parameter Binding**: The query builder collects named parameters separately from the SQL string, ensuring safe parameterization.

## Real World Context

Every ORM and framework (Laravel, Doctrine DBAL, CakePHP) uses query builders internally. Understanding how they work lets you debug generated queries, extend them with custom methods, and build domain-specific query APIs.

## Deep Dive

An immutable query builder clones itself on every method call:

```php
<?php
class QueryBuilder {
    private string $table = '';
    private array $columns = ['*'];
    private array $wheres = [];
    private array $bindings = [];
    private array $orderBy = [];
    private ?int $limit = null;
    private ?int $offset = null;
    
    public function __construct(private PDO $pdo) {}
    
    public function table(string $table): self {
        $clone = clone $this;
        $clone->table = $table;
        return $clone;
    }
    
    public function select(string ...$columns): self {
        $clone = clone $this;
        $clone->columns = $columns;
        return $clone;
    }
    
    public function where(string $column, mixed $value, string $op = '='): self {
        $clone = clone $this;
        $placeholder = ':where_' . count($clone->wheres);
        $clone->wheres[] = "$column $op $placeholder";
        $clone->bindings[$placeholder] = $value;
        return $clone;
    }
    
    public function orderBy(string $column, string $dir = 'ASC'): self {
        $clone = clone $this;
        $dir = strtoupper($dir) === 'DESC' ? 'DESC' : 'ASC';
        $clone->orderBy[] = "$column $dir";
        return $clone;
    }
    
    public function limit(int $limit): self {
        $clone = clone $this;
        $clone->limit = $limit;
        return $clone;
    }
    
    public function offset(int $offset): self {
        $clone = clone $this;
        $clone->offset = $offset;
        return $clone;
    }
    
    public function toSql(): string {
        $sql = 'SELECT ' . implode(', ', $this->columns);
        $sql .= ' FROM ' . $this->table;
        if ($this->wheres) {
            $sql .= ' WHERE ' . implode(' AND ', $this->wheres);
        }
        if ($this->orderBy) {
            $sql .= ' ORDER BY ' . implode(', ', $this->orderBy);
        }
        if ($this->limit !== null) {
            $sql .= ' LIMIT ' . $this->limit;
        }
        if ($this->offset !== null) {
            $sql .= ' OFFSET ' . $this->offset;
        }
        return $sql;
    }
    
    public function get(): array {
        $stmt = $this->pdo->prepare($this->toSql());
        $stmt->execute($this->bindings);
        return $stmt->fetchAll();
    }
    
    public function first(): ?array {
        return $this->limit(1)->get()[0] ?? null;
    }
}
```

Usage is clean and readable:

```php
<?php
$qb = new QueryBuilder($pdo);

$users = $qb
    ->table('users')
    ->select('id', 'name', 'email')
    ->where('status', 'active')
    ->where('created_at', '2024-01-01', '>=')
    ->orderBy('name')
    ->limit(10)
    ->get();
```

The generated SQL uses named parameters, completely preventing SQL injection.

With PHP 8.5's pipe operator `|>`, you can chain result processing after the query:

```php
<?php
// PHP 8.5 pipe operator for processing results
$activeEmails = $qb
    ->table('users')
    ->select('email')
    ->where('status', 'active')
    ->get()
    |> (fn($rows) => array_column($rows, 'email'))
    |> (fn($emails) => array_map(strtolower(...), $emails));
```

The pipe operator passes the left-hand result as the sole argument to the right-hand callable, making post-query transformations more readable.

## Common Pitfalls

1. **Mutating the builder instead of cloning** — If `where()` modifies `$this`, reusing a base query accidentally accumulates conditions across unrelated queries.
2. **Interpolating column/table names from user input** — The query builder parameterizes values, but column and table names cannot be parameterized. Always validate them against an allowlist.

## Best Practices

1. **Always clone in each builder method** — Immutability lets you safely derive multiple queries from a common base without side effects.
2. **Expose `toSql()` for debugging** — Being able to inspect the generated SQL makes it easy to diagnose performance issues with EXPLAIN.

## Summary

- Query builders provide a fluent interface that generates safe, parameterized SQL.
- Immutable builders (clone-on-modify) prevent accidental state sharing.
- PHP 8.5's pipe operator `|>` lets you chain result transformations fluently after queries.
- Always validate column and table names against an allowlist since they cannot be parameterized.

## Code Examples

**Immutable query builder with PHP 8.5 pipe operator**

```php
<?php
declare(strict_types=1);

$qb = new QueryBuilder($pdo);

// Base query
$base = $qb->table('users')->select('id', 'name', 'email');

// Derived queries (safe because immutable)
$active = $base->where('status', 'active');
$admins = $base->where('role', 'admin');

// These are independent — $active doesn't have the role filter
$activeUsers = $active->orderBy('name')->limit(20)->get();
$adminUsers = $admins->orderBy('created_at', 'DESC')->get();

// PHP 8.5 pipe operator for post-processing
$emails = $active->get()
    |> (fn($rows) => array_column($rows, 'email'));
?>
```


## Resources

- [Fluent Interface Pattern](https://en.wikipedia.org/wiki/Fluent_interface) — Fluent interface design pattern overview

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*