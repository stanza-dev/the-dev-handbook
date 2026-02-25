---
source_course: "php-performance"
source_lesson: "php-performance-connection-pooling"
---

# Connection Management

## Introduction
Database connections are expensive. Manage them wisely.

## Key Concepts
- **Persistent Connections**: Reusing database connections across requests with `PDO::ATTR_PERSISTENT`, avoiding TCP/SSL handshake overhead.
- **Connection Pooling**: A pool of pre-established database connections shared among application processes.
- **Connection Limits**: MySQL's `max_connections` setting and PHP-FPM's `pm.max_children` must be balanced to prevent exhaustion.
- **Connection Overhead**: Each new MySQL connection involves TCP handshake, SSL negotiation, and authentication — 10-50ms per connection.

## Real World Context
Database connection overhead becomes significant at scale. If each of your 100 PHP-FPM workers opens a new MySQL connection per request, you're creating and destroying hundreds of connections per second. Persistent connections or a connection pooler like ProxySQL can reduce connection overhead by 90% and prevent `Too many connections` errors.

## Deep Dive
### Intro

Database connections are expensive. Manage them wisely.

### Connection pooling

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

### Persistent connections

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

### Connection timeouts

```php
<?php
$pdo = new PDO($dsn, $user, $pass, [
    PDO::ATTR_TIMEOUT => 5,  // 5 second timeout
]);

// For MySQL specifically
$pdo->exec('SET wait_timeout = 28800');  // 8 hours
$pdo->exec('SET interactive_timeout = 28800');
```

### Read replicas

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

## Common Pitfalls
1. **Too many persistent connections** — Each PHP-FPM worker holds a persistent connection. With 100 workers × 3 database servers, you need 300 MySQL connections. Plan capacity carefully.
2. **Not closing connections in long-running scripts** — Queue workers and daemons should periodically reconnect to prevent stale connections from holding resources.

## Best Practices
1. **Use persistent connections in production** — Set `PDO::ATTR_PERSISTENT => true` and configure MySQL `max_connections` to accommodate your PHP-FPM worker count.
2. **Consider ProxySQL for large deployments** — ProxySQL provides connection multiplexing, query caching, and read/write splitting without code changes.

## Summary
- Persistent connections eliminate the 10-50ms overhead of establishing new database connections per request.
- Balance `max_connections` with your PHP-FPM worker count to prevent connection exhaustion.
- Use ProxySQL for connection pooling in large-scale deployments.

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


## Resources

- [PDO Connections](https://www.php.net/manual/en/pdo.connections.php) — PHP PDO connection management

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*