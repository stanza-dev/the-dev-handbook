---
source_course: "php-databases"
source_lesson: "php-databases-query-builder-basics"
---

# Building a Query Builder

Query builders provide a fluent interface for constructing SQL queries safely.

## Basic Query Builder

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
    
    public function __construct(
        private PDO $pdo
    ) {}
    
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
    
    public function where(string $column, mixed $value, string $operator = '='): self {
        $clone = clone $this;
        $placeholder = ':where_' . count($clone->wheres);
        $clone->wheres[] = "$column $operator $placeholder";
        $clone->bindings[$placeholder] = $value;
        return $clone;
    }
    
    public function orderBy(string $column, string $direction = 'ASC'): self {
        $clone = clone $this;
        $direction = strtoupper($direction) === 'DESC' ? 'DESC' : 'ASC';
        $clone->orderBy[] = "$column $direction";
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
        $result = $this->limit(1)->get();
        return $result[0] ?? null;
    }
}
```

## Usage

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

// Generated SQL:
// SELECT id, name, email FROM users 
// WHERE status = :where_0 AND created_at >= :where_1
// ORDER BY name ASC LIMIT 10
```

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*