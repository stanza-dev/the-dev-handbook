---
source_course: "php-performance"
source_lesson: "php-performance-cache-stampede"
---

# Preventing Cache Stampede

## Introduction
A cache stampede (thundering herd) occurs when many requests simultaneously find an expired cache entry and all try to regenerate it.

## Key Concepts
- **Cache Stampede**: When a popular cache entry expires, many concurrent requests simultaneously query the database to rebuild it.
- **Locking (Mutex)**: Only one process rebuilds the cache while others wait or serve stale data.
- **Probabilistic Early Expiration**: Randomly refreshing cache entries before they expire to prevent synchronized expiration.
- **Stale-While-Revalidate**: Serving the stale cached value while asynchronously refreshing in the background.

## Real World Context
Cache stampedes have caused major outages at companies like Facebook (the Thundering Herd problem). When a frequently accessed cache key expires and 10,000 requests simultaneously try to rebuild it, the database can be overwhelmed. This is especially dangerous for expensive queries (aggregations, joins) that take seconds to compute.

## Deep Dive
### Intro

A cache stampede (thundering herd) occurs when many requests simultaneously find an expired cache entry and all try to regenerate it.

### The problem

```php
<?php
// Scenario: Popular product page, cache expires
// 1000 concurrent users hit the page
// All 1000 trigger expensive database query!

function getProduct(int $id): array
{
    $cached = $cache->get("product:$id");
    
    if ($cached === null) {
        // STAMPEDE: All requests execute this simultaneously!
        $product = $this->expensiveQuery($id);
        $cache->set("product:$id", $product, 3600);
    }
    
    return $cached;
}
```

### Solution 1: locking

```php
<?php
class StampedeProtectedCache
{
    public function __construct(
        private CacheInterface $cache,
        private LockFactory $locks
    ) {}
    
    public function remember(string $key, int $ttl, callable $callback): mixed
    {
        $value = $this->cache->get($key);
        
        if ($value !== null) {
            return $value;
        }
        
        // Try to acquire lock
        $lock = $this->locks->createLock($key . ':lock', 30);
        
        if ($lock->acquire()) {
            try {
                // Double-check after acquiring lock
                $value = $this->cache->get($key);
                if ($value !== null) {
                    return $value;
                }
                
                // Generate and cache
                $value = $callback();
                $this->cache->set($key, $value, $ttl);
                return $value;
                
            } finally {
                $lock->release();
            }
        }
        
        // Couldn't get lock - wait for cached value
        return $this->waitForValue($key, 5);
    }
    
    private function waitForValue(string $key, int $maxWait): mixed
    {
        $start = time();
        
        while (time() - $start < $maxWait) {
            $value = $this->cache->get($key);
            if ($value !== null) {
                return $value;
            }
            usleep(100000); // 100ms
        }
        
        throw new RuntimeException('Cache value not available');
    }
}
```

### Solution 2: probabilistic early expiration

```php
<?php
class XFetchCache
{
    public function get(string $key, callable $callback, int $ttl): mixed
    {
        $cached = $this->cache->get($key);
        
        if ($cached !== null) {
            $data = unserialize($cached);
            
            // Probabilistic early recomputation
            $delta = $data['delta'];
            $expiry = $data['expiry'];
            $now = time();
            
            // XFetch algorithm: recompute early based on probability
            $random = -$delta * log(random_int(1, 1000) / 1000);
            
            if ($now + $random >= $expiry) {
                // This request will regenerate cache
                return $this->regenerate($key, $callback, $ttl);
            }
            
            return $data['value'];
        }
        
        return $this->regenerate($key, $callback, $ttl);
    }
    
    private function regenerate(string $key, callable $callback, int $ttl): mixed
    {
        $start = microtime(true);
        $value = $callback();
        $delta = microtime(true) - $start;
        
        $data = [
            'value' => $value,
            'delta' => $delta,
            'expiry' => time() + $ttl,
        ];
        
        $this->cache->set($key, serialize($data), $ttl + 60);
        
        return $value;
    }
}
```

### Solution 3: background refresh

```php
<?php
class BackgroundRefreshCache
{
    public function get(string $key, callable $callback, int $ttl): mixed
    {
        $data = $this->cache->get($key);
        
        if ($data !== null) {
            $meta = $this->cache->get($key . ':meta');
            
            // Check if we should background refresh (e.g., 80% through TTL)
            if ($meta && time() > $meta['refresh_at']) {
                $this->queueRefresh($key, $callback, $ttl);
            }
            
            return $data; // Return stale data immediately
        }
        
        // Cache miss - must regenerate synchronously
        return $this->regenerate($key, $callback, $ttl);
    }
    
    private function queueRefresh(string $key, callable $callback, int $ttl): void
    {
        // Don't queue if already queued
        if ($this->cache->get($key . ':refreshing')) {
            return;
        }
        
        $this->cache->set($key . ':refreshing', true, 60);
        
        // Queue background job
        $this->queue->push('cache:refresh', [
            'key' => $key,
            'callback' => serialize($callback),
            'ttl' => $ttl,
        ]);
    }
}
```

## Common Pitfalls
1. **No protection against thundering herd** — Without locking or early expiration, every popular cache entry is a potential stampede waiting to happen.
2. **Using fixed TTL for all entries** — All entries with the same TTL expire at the same time, causing mass stampedes. Add random jitter to TTL values.

## Best Practices
1. **Implement cache locking** — Use Redis `SET key value NX EX 30` as a distributed mutex so only one process rebuilds the cache.
2. **Add TTL jitter** — Instead of `TTL = 3600`, use `TTL = 3600 + random(0, 300)` to spread expiration times and prevent synchronized stampedes.

## Summary
- Cache stampedes occur when a popular cache entry expires and many requests simultaneously try to rebuild it.
- Use cache locking (mutex), probabilistic early expiration, or stale-while-revalidate to prevent stampedes.
- Add random jitter to TTL values to prevent mass synchronized expiration.

## Resources

- [Cache Stampede](https://en.wikipedia.org/wiki/Cache_stampede) — Understanding cache stampede problem

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*