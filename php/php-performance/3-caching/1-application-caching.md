---
source_course: "php-performance"
source_lesson: "php-performance-application-caching"
---

# Application-Level Caching

## Introduction
Caching stores computed results to avoid redundant processing.

## Key Concepts
- **Cache-Aside Pattern**: Application checks cache first, falls back to database on miss, then populates cache.
- **TTL (Time To Live)**: Maximum age of a cached entry before it's considered stale and must be refreshed.
- **Cache Invalidation**: Removing or updating cached data when the underlying source data changes.
- **Serialization**: Converting PHP objects to storable strings (e.g., `serialize()`, `json_encode()`, `igbinary`).

## Real World Context
Caching is the most impactful performance optimization for read-heavy PHP applications. Stack Overflow caches aggressively and serves millions of requests per day from just a few servers. A well-implemented cache layer can reduce database load by 90% and cut response times from hundreds of milliseconds to single digits.

## Deep Dive
### Intro

Caching stores computed results to avoid redundant processing.

### Apcu (in-memory cache)

```php
<?php
// Store in cache
apcu_store('user_123', $userData, 3600);  // 1 hour TTL

// Retrieve from cache
$cached = apcu_fetch('user_123', $success);
if ($success) {
    return $cached;
}

// Delete from cache
apcu_delete('user_123');

// Check existence
if (apcu_exists('user_123')) {
    // ...
}
```

### Cache-aside pattern

```php
<?php
class UserRepository
{
    public function find(int $id): ?User
    {
        $key = "user:$id";
        
        // Try cache first
        $cached = apcu_fetch($key, $success);
        if ($success) {
            return $cached;
        }
        
        // Cache miss - fetch from database
        $user = $this->fetchFromDatabase($id);
        
        if ($user) {
            apcu_store($key, $user, 3600);
        }
        
        return $user;
    }
    
    public function update(User $user): void
    {
        $this->saveToDatabase($user);
        
        // Invalidate cache
        apcu_delete("user:{$user->id}");
    }
}
```

### Redis caching

```php
<?php
class RedisCache
{
    private Redis $redis;
    
    public function __construct(string $host = '127.0.0.1', int $port = 6379)
    {
        $this->redis = new Redis();
        $this->redis->connect($host, $port);
    }
    
    public function get(string $key): mixed
    {
        $value = $this->redis->get($key);
        return $value !== false ? unserialize($value) : null;
    }
    
    public function set(string $key, mixed $value, int $ttl = 3600): void
    {
        $this->redis->setex($key, $ttl, serialize($value));
    }
    
    public function delete(string $key): void
    {
        $this->redis->del($key);
    }
    
    public function remember(string $key, int $ttl, callable $callback): mixed
    {
        $cached = $this->get($key);
        
        if ($cached !== null) {
            return $cached;
        }
        
        $value = $callback();
        $this->set($key, $value, $ttl);
        
        return $value;
    }
}

// Usage
$cache = new RedisCache();

$users = $cache->remember('active_users', 300, function() use ($repo) {
    return $repo->findAllActive();
});
```

### Cache tags (invalidation groups)

```php
<?php
class TaggedCache
{
    private Redis $redis;
    
    public function set(string $key, mixed $value, int $ttl, array $tags = []): void
    {
        $this->redis->setex($key, $ttl, serialize($value));
        
        // Track keys by tag
        foreach ($tags as $tag) {
            $this->redis->sAdd("tag:$tag", $key);
        }
    }
    
    public function invalidateTag(string $tag): void
    {
        $keys = $this->redis->sMembers("tag:$tag");
        
        if ($keys) {
            $this->redis->del(...$keys);
            $this->redis->del("tag:$tag");
        }
    }
}

// Usage
$cache->set('user:1', $user1, 3600, ['users', 'user:1']);
$cache->set('user:2', $user2, 3600, ['users', 'user:2']);
$cache->set('user_list', $allUsers, 3600, ['users']);

// Invalidate all user-related cache
$cache->invalidateTag('users');
```

## Common Pitfalls
1. **Setting TTL too high** — Long TTLs mean users see stale data. Balance freshness and performance based on how often your data changes.
2. **Not handling cache failures gracefully** — When Redis/Memcached goes down, your app should fall back to the database, not crash. Use try/catch around cache operations.

## Best Practices
1. **Use cache tags for related invalidation** — When a product changes, invalidate all cache entries tagged with that product ID instead of clearing everything.
2. **Cache at the right granularity** — Cache computed results (e.g., rendered HTML fragments, aggregated stats) not raw database rows, to maximize the benefit.

## Summary
- The cache-aside pattern (check cache → fallback to DB → populate cache) is the most common caching strategy.
- Balance TTL between data freshness and performance based on your data's change frequency.
- Use cache tags and event-driven invalidation for precise cache management.

## Resources

- [APCu](https://www.php.net/manual/en/book.apcu.php) — PHP APCu extension documentation

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*