---
source_course: "php-performance"
source_lesson: "php-performance-identifying-bottlenecks"
---

# Identifying Performance Bottlenecks

## Introduction
Most performance issues fall into predictable categories. Learn to recognize them.

## Key Concepts
- **N+1 Query Problem**: Executing one query per item instead of a single batch query — the most common PHP performance killer.
- **I/O vs CPU Bottlenecks**: I/O-bound code (database, network, disk) is usually the primary bottleneck; CPU-bound code (computation) is less common.
- **Sequential vs Parallel I/O**: Sequential API calls or file reads that could be parallelized with promises or multi-curl.
- **Memory Bloat**: Loading entire datasets into memory instead of using generators or streaming.

## Real World Context
Most PHP performance issues are I/O bound — waiting for database queries, external APIs, or file reads. The N+1 problem alone can turn a 50ms page into a 5-second page when loading related records. Fixing N+1 queries often yields 10-100x improvements with minimal code changes.

## Deep Dive
### Intro

Most performance issues fall into predictable categories. Learn to recognize them.

### Common bottlenecks

### 1. Database Queries

```php
<?php
// Enable query logging
$pdo->setAttribute(PDO::ATTR_STATEMENT_CLASS, [LoggedStatement::class]);

class LoggedStatement extends PDOStatement
{
    private float $startTime;
    
    public function execute(?array $params = null): bool
    {
        $this->startTime = microtime(true);
        $result = parent::execute($params);
        
        $duration = (microtime(true) - $this->startTime) * 1000;
        
        if ($duration > 100) {  // Slow query threshold
            error_log(sprintf(
                'Slow query (%.2fms): %s',
                $duration,
                $this->queryString
            ));
        }
        
        return $result;
    }
}
```

### 2. N+1 Queries

```php
<?php
// BAD: N+1 - 101 queries for 100 users
$users = $pdo->query('SELECT * FROM users LIMIT 100');
foreach ($users as $user) {
    $stmt = $pdo->prepare('SELECT * FROM orders WHERE user_id = ?');
    $stmt->execute([$user['id']]);
}

// GOOD: 2 queries
$users = $pdo->query('SELECT * FROM users LIMIT 100');
$userIds = array_column($users, 'id');
$orders = $pdo->query('SELECT * FROM orders WHERE user_id IN (' . implode(',', $userIds) . ')');
```

### 3. Memory Bloat

```php
<?php
// BAD: Loads entire file into memory
$content = file_get_contents('huge_file.csv');
$lines = explode("\n", $content);

// GOOD: Stream line by line
$handle = fopen('huge_file.csv', 'r');
while (($line = fgets($handle)) !== false) {
    processLine($line);
}
fclose($handle);
```

### 4. External API Calls

```php
<?php
// BAD: Sequential calls
$user = fetchFromApi('/users/1');      // 200ms
$orders = fetchFromApi('/orders');      // 300ms
$products = fetchFromApi('/products');  // 250ms
// Total: 750ms

// GOOD: Parallel with curl_multi or Guzzle
$responses = $guzzle->request([
    ['GET', '/users/1'],
    ['GET', '/orders'],
    ['GET', '/products'],
]);
// Total: ~300ms (slowest request)
```

### Performance checklist

```php
<?php
class PerformanceChecker
{
    public function analyze(): array
    {
        $issues = [];
        
        // Check OPcache
        if (!function_exists('opcache_get_status') || !opcache_get_status()) {
            $issues[] = 'OPcache is disabled';
        }
        
        // Check memory limit
        $memLimit = ini_get('memory_limit');
        if ($this->parseBytes($memLimit) < 128 * 1024 * 1024) {
            $issues[] = "Low memory limit: $memLimit";
        }
        
        // Check realpath cache
        $realpathSize = ini_get('realpath_cache_size');
        if ($this->parseBytes($realpathSize) < 4 * 1024 * 1024) {
            $issues[] = "Small realpath cache: $realpathSize";
        }
        
        return $issues;
    }
}
```

## Common Pitfalls
1. **Ignoring the N+1 query problem** — ORMs like Doctrine and Eloquent make N+1 easy to introduce accidentally. Use eager loading (`JOIN` or `WHERE IN`) for related data.
2. **Making sequential API calls** — If you need data from 5 external APIs, send all requests concurrently with `curl_multi_*` or Guzzle promises instead of waiting for each one.

## Best Practices
1. **Enable database query logging in development** — Monitor query counts per page load. A sudden jump from 5 to 50 queries signals an N+1 problem.
2. **Use batch operations** — Replace loops of individual INSERT/UPDATE statements with bulk operations (e.g., `INSERT INTO ... VALUES (...), (...), (...)`).

## Summary
- N+1 queries are the most common PHP bottleneck — fix with eager loading and batch queries.
- Most PHP performance issues are I/O bound, not CPU bound.
- Parallelize independent I/O operations with promises or multi-curl.

## Code Examples

**Performance monitoring middleware**

```php
<?php
declare(strict_types=1);

// Performance monitoring middleware
class PerformanceMiddleware
{
    private float $requestStart;
    private array $metrics = [];
    
    public function before(): void
    {
        $this->requestStart = microtime(true);
        $this->metrics['memory_start'] = memory_get_usage();
        $this->metrics['queries'] = 0;
        $this->metrics['cache_hits'] = 0;
        $this->metrics['cache_misses'] = 0;
    }
    
    public function recordQuery(float $duration): void
    {
        $this->metrics['queries']++;
        $this->metrics['query_time'] = ($this->metrics['query_time'] ?? 0) + $duration;
    }
    
    public function recordCacheHit(): void
    {
        $this->metrics['cache_hits']++;
    }
    
    public function recordCacheMiss(): void
    {
        $this->metrics['cache_misses']++;
    }
    
    public function after(): void
    {
        $duration = (microtime(true) - $this->requestStart) * 1000;
        $memory = (memory_get_usage() - $this->metrics['memory_start']) / 1024 / 1024;
        
        // Add performance headers
        header(sprintf('X-Response-Time: %.2fms', $duration));
        header(sprintf('X-Memory-Usage: %.2fMB', $memory));
        header(sprintf('X-Query-Count: %d', $this->metrics['queries']));
        
        // Log slow requests
        if ($duration > 500) {
            $this->logSlowRequest($duration);
        }
        
        // Send to monitoring
        $this->sendMetrics($duration, $memory);
    }
    
    private function logSlowRequest(float $duration): void
    {
        $data = [
            'uri' => $_SERVER['REQUEST_URI'],
            'method' => $_SERVER['REQUEST_METHOD'],
            'duration_ms' => $duration,
            'queries' => $this->metrics['queries'],
            'query_time_ms' => $this->metrics['query_time'] ?? 0,
            'memory_mb' => (memory_get_peak_usage() / 1024 / 1024),
            'timestamp' => date('c'),
        ];
        
        error_log('SLOW_REQUEST: ' . json_encode($data));
    }
    
    private function sendMetrics(float $duration, float $memory): void
    {
        // Send to StatsD, Prometheus, etc.
    }
}

// Usage in application
$perf = new PerformanceMiddleware();
$perf->before();

// Application code...

$perf->after();
?>
```


## Resources

- [Xdebug Profiler](https://xdebug.org/docs/profiler) — Xdebug profiling documentation

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*