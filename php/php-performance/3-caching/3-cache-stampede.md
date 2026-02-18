---
source_course: "php-performance"
source_lesson: "php-performance-cache-stampede"
---

# Preventing Cache Stampede

A cache stampede (thundering herd) occurs when many requests simultaneously find an expired cache entry and all try to regenerate it.

## The Problem

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

## Solution 1: Locking

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

## Solution 2: Probabilistic Early Expiration

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

## Solution 3: Background Refresh

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

## Resources

- [Cache Stampede](https://en.wikipedia.org/wiki/Cache_stampede) — Understanding cache stampede problem

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*