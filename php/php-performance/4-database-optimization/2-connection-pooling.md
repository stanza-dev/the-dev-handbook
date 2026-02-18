---
source_course: "php-performance"
source_lesson: "php-performance-connection-pooling"
---

# Connection Management

Database connections are expensive. Manage them wisely.

## Connection Pooling

```php
<?php
// Singleton pattern for connection reuse
class Database
{
    private static ?PDO $instance = null;
    
    public static function getConnection(): PDO
    {
        if (self::$instance === null) {
            self::$instance = new PDO(
                $_ENV['DATABASE_URL'],
                $_ENV['DATABASE_USER'],
                $_ENV['DATABASE_PASS'],
                [
                    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
                    PDO::ATTR_PERSISTENT => true,  // Persistent connection
                ]
            );
        }
        
        return self::$instance;
    }
}
```

## Persistent Connections

```php
<?php
// Enable persistent connections
$pdo = new PDO($dsn, $user, $pass, [
    PDO::ATTR_PERSISTENT => true,
]);

// Benefits:
// - Avoids connection overhead per request
// - Reuses existing connections

// Caveats:
// - Connections persist across requests
// - Need to clean up state (temp tables, locks)
// - May need to increase max_connections
```

## Connection Timeouts

```php
<?php
$pdo = new PDO($dsn, $user, $pass, [
    PDO::ATTR_TIMEOUT => 5,  // 5 second timeout
]);

// For MySQL specifically
$pdo->exec('SET wait_timeout = 28800');  // 8 hours
$pdo->exec('SET interactive_timeout = 28800');
```

## Read Replicas

```php
<?php
class DatabaseManager
{
    private PDO $writer;
    private array $readers = [];
    
    public function __construct(string $writerDsn, array $readerDsns)
    {
        $this->writer = new PDO($writerDsn);
        
        foreach ($readerDsns as $dsn) {
            $this->readers[] = new PDO($dsn);
        }
    }
    
    public function write(): PDO
    {
        return $this->writer;
    }
    
    public function read(): PDO
    {
        // Round-robin or random selection
        return $this->readers[array_rand($this->readers)];
    }
}

// Usage
$db = new DatabaseManager(
    'mysql:host=primary.db;dbname=app',
    [
        'mysql:host=replica1.db;dbname=app',
        'mysql:host=replica2.db;dbname=app',
    ]
);

// Reads go to replicas (scalable)
$users = $db->read()->query('SELECT * FROM users');

// Writes go to primary
$db->write()->exec('INSERT INTO users ...');
```

## Code Examples

**Optimized query builder with index hints**

```php
<?php
declare(strict_types=1);

// Query builder with automatic optimization hints
class OptimizedQueryBuilder
{
    private PDO $pdo;
    private string $table;
    private array $select = ['*'];
    private array $where = [];
    private array $bindings = [];
    private ?int $limit = null;
    private ?int $offset = null;
    private array $orderBy = [];
    private bool $forceIndex = false;
    private ?string $indexHint = null;
    
    public function __construct(PDO $pdo, string $table)
    {
        $this->pdo = $pdo;
        $this->table = $table;
    }
    
    public function select(string ...$columns): self
    {
        $this->select = $columns ?: ['*'];
        return $this;
    }
    
    public function where(string $column, mixed $value, string $operator = '='): self
    {
        $param = ':w' . count($this->bindings);
        $this->where[] = "$column $operator $param";
        $this->bindings[$param] = $value;
        return $this;
    }
    
    public function useIndex(string $indexName): self
    {
        $this->indexHint = "USE INDEX ($indexName)";
        return $this;
    }
    
    public function forceIndex(string $indexName): self
    {
        $this->indexHint = "FORCE INDEX ($indexName)";
        return $this;
    }
    
    public function limit(int $limit, ?int $offset = null): self
    {
        $this->limit = $limit;
        $this->offset = $offset;
        return $this;
    }
    
    public function orderBy(string $column, string $direction = 'ASC'): self
    {
        $this->orderBy[] = "$column $direction";
        return $this;
    }
    
    public function toSql(): string
    {
        $sql = 'SELECT ' . implode(', ', $this->select);
        $sql .= ' FROM ' . $this->table;
        
        if ($this->indexHint) {
            $sql .= ' ' . $this->indexHint;
        }
        
        if ($this->where) {
            $sql .= ' WHERE ' . implode(' AND ', $this->where);
        }
        
        if ($this->orderBy) {
            $sql .= ' ORDER BY ' . implode(', ', $this->orderBy);
        }
        
        if ($this->limit !== null) {
            $sql .= ' LIMIT ' . $this->limit;
            if ($this->offset !== null) {
                $sql .= ' OFFSET ' . $this->offset;
            }
        }
        
        return $sql;
    }
    
    public function explain(): array
    {
        $stmt = $this->pdo->prepare('EXPLAIN ' . $this->toSql());
        $stmt->execute($this->bindings);
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
    
    public function get(): array
    {
        $stmt = $this->pdo->prepare($this->toSql());
        $stmt->execute($this->bindings);
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
}

// Usage
$query = new OptimizedQueryBuilder($pdo, 'orders');

$orders = $query
    ->select('id', 'total', 'created_at')
    ->where('user_id', 123)
    ->where('status', 'completed')
    ->forceIndex('idx_user_status')  // Force specific index
    ->orderBy('created_at', 'DESC')
    ->limit(10)
    ->get();

// Debug: check execution plan
print_r($query->explain());
?>
```


---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*