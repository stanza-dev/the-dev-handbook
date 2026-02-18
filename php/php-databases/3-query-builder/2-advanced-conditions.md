---
source_course: "php-databases"
source_lesson: "php-databases-advanced-conditions"
---

# Advanced Query Conditions

Extend the query builder with complex condition support.

## IN and NOT IN

```php
<?php
class QueryBuilder {
    // ... previous code ...
    
    public function whereIn(string $column, array $values): self {
        if (empty($values)) {
            // No values = no results
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
    
    public function whereNotIn(string $column, array $values): self {
        if (empty($values)) {
            return $this;  // No exclusions = all results
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
}
```

## NULL Checks

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

## OR Conditions

```php
<?php
public function orWhere(string $column, mixed $value, string $operator = '='): self {
    $clone = clone $this;
    $placeholder = ':orwhere_' . count($clone->wheres);
    
    if (empty($clone->wheres)) {
        $clone->wheres[] = "$column $operator $placeholder";
    } else {
        // Replace last AND with OR group
        $lastWhere = array_pop($clone->wheres);
        $clone->wheres[] = "($lastWhere OR $column $operator $placeholder)";
    }
    
    $clone->bindings[$placeholder] = $value;
    return $clone;
}

// Better: Grouped conditions
public function whereGroup(callable $callback): self {
    $clone = clone $this;
    $group = new QueryBuilder($this->pdo);
    $callback($group);
    
    if ($group->wheres) {
        $clone->wheres[] = '(' . implode(' AND ', $group->wheres) . ')';
        $clone->bindings = array_merge($clone->bindings, $group->bindings);
    }
    
    return $clone;
}
```

## LIKE Queries

```php
<?php
public function whereLike(string $column, string $pattern): self {
    $clone = clone $this;
    $placeholder = ':like_' . count($clone->wheres);
    $clone->wheres[] = "$column LIKE $placeholder";
    $clone->bindings[$placeholder] = $pattern;
    return $clone;
}

// Usage
$users = $qb
    ->table('users')
    ->whereLike('email', '%@gmail.com')
    ->get();
```

## Code Examples

**Complete query builder implementation**

```php
<?php
declare(strict_types=1);

// Complete query builder with all features
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
    
    public function select(string ...$columns): self {
        $clone = clone $this;
        $clone->columns = $columns ?: ['*'];
        return $clone;
    }
    
    public function join(string $table, string $first, string $operator, string $second): self {
        $clone = clone $this;
        $clone->joins[] = "JOIN $table ON $first $operator $second";
        return $clone;
    }
    
    public function leftJoin(string $table, string $first, string $operator, string $second): self {
        $clone = clone $this;
        $clone->joins[] = "LEFT JOIN $table ON $first $operator $second";
        return $clone;
    }
    
    public function where(string $column, mixed $value, string $operator = '='): self {
        $clone = clone $this;
        $key = ':w' . count($clone->bindings);
        $clone->wheres[] = ['type' => 'AND', 'clause' => "$column $operator $key"];
        $clone->bindings[$key] = $value;
        return $clone;
    }
    
    public function orWhere(string $column, mixed $value, string $operator = '='): self {
        $clone = clone $this;
        $key = ':w' . count($clone->bindings);
        $clone->wheres[] = ['type' => 'OR', 'clause' => "$column $operator $key"];
        $clone->bindings[$key] = $value;
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
        $clone->wheres[] = ['type' => 'AND', 'clause' => "$column IN (" . implode(', ', $keys) . ')'];
        return $clone;
    }
    
    public function whereRaw(string $sql): self {
        $clone = clone $this;
        $clone->wheres[] = ['type' => 'AND', 'clause' => $sql];
        return $clone;
    }
    
    public function orderBy(string $column, string $direction = 'ASC'): self {
        $clone = clone $this;
        $clone->orderBy[] = $column . ' ' . (strtoupper($direction) === 'DESC' ? 'DESC' : 'ASC');
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
        $sql = 'SELECT ' . implode(', ', $this->columns) . ' FROM ' . $this->table;
        
        if ($this->joins) {
            $sql .= ' ' . implode(' ', $this->joins);
        }
        
        if ($this->wheres) {
            $sql .= ' WHERE ' . $this->buildWhereClause();
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
    
    private function buildWhereClause(): string {
        $parts = [];
        foreach ($this->wheres as $i => $where) {
            if ($i === 0) {
                $parts[] = $where['clause'];
            } else {
                $parts[] = $where['type'] . ' ' . $where['clause'];
            }
        }
        return implode(' ', $parts);
    }
    
    public function get(): array {
        $stmt = $this->pdo->prepare($this->toSql());
        $stmt->execute($this->bindings);
        return $stmt->fetchAll();
    }
    
    public function first(): ?array {
        return $this->limit(1)->get()[0] ?? null;
    }
    
    public function count(): int {
        $clone = clone $this;
        $clone->columns = ['COUNT(*) as count'];
        $clone->orderBy = [];
        $clone->limit = null;
        $clone->offset = null;
        
        $result = $clone->first();
        return (int) ($result['count'] ?? 0);
    }
}
?>
```


---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*