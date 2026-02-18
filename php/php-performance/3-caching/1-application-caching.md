---
source_course: "php-performance"
source_lesson: "php-performance-application-caching"
---

# Application-Level Caching

Caching stores computed results to avoid redundant processing.

## APCu (In-Memory Cache)

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

## Cache-Aside Pattern

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

## Redis Caching

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

## Cache Tags (Invalidation Groups)

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

## Resources

- [APCu](https://www.php.net/manual/en/book.apcu.php) — PHP APCu extension documentation

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*