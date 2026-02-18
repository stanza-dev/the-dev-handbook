---
source_course: "php-performance"
source_lesson: "php-performance-horizontal-scaling"
---

# Horizontal Scaling Strategies

Scale out by adding more servers rather than bigger servers.

## Session Management

```php
<?php
// Problem: Sessions stored locally don't work with multiple servers

// Solution 1: Sticky sessions (load balancer affinity)
// - Same user always goes to same server
// - Simple but limits flexibility

// Solution 2: Centralized session storage (Redis)
ini_set('session.save_handler', 'redis');
ini_set('session.save_path', 'tcp://redis.example.com:6379');

// Or programmatically
class RedisSessionHandler implements SessionHandlerInterface
{
    private Redis $redis;
    private int $ttl = 3600;
    
    public function __construct(Redis $redis)
    {
        $this->redis = $redis;
    }
    
    public function open(string $path, string $name): bool
    {
        return true;
    }
    
    public function close(): bool
    {
        return true;
    }
    
    public function read(string $id): string|false
    {
        return $this->redis->get("session:$id") ?: '';
    }
    
    public function write(string $id, string $data): bool
    {
        return $this->redis->setex("session:$id", $this->ttl, $data);
    }
    
    public function destroy(string $id): bool
    {
        return $this->redis->del("session:$id") > 0;
    }
    
    public function gc(int $max_lifetime): int|false
    {
        return 0;  // Redis handles expiration
    }
}

$handler = new RedisSessionHandler(new Redis());
session_set_save_handler($handler, true);
```

## Shared Nothing Architecture

```php
<?php
// Each request is independent
// - No local file storage
// - No local session storage
// - No shared memory between requests

// Store uploads in object storage (S3)
class S3FileStorage implements FileStorage
{
    public function store(string $localPath, string $remotePath): string
    {
        $this->s3->putObject([
            'Bucket' => $this->bucket,
            'Key' => $remotePath,
            'SourceFile' => $localPath,
        ]);
        
        return $this->getPublicUrl($remotePath);
    }
}

// Store cache in Redis (shared across servers)
class DistributedCache
{
    public function __construct(private Redis $redis) {}
    
    public function get(string $key): mixed { /* ... */ }
    public function set(string $key, mixed $value, int $ttl = 3600): void { /* ... */ }
}
```

## Database Scaling

```php
<?php
// Read replicas for read scaling
class DatabaseCluster
{
    private PDO $master;
    private array $replicas = [];
    
    public function master(): PDO
    {
        return $this->master;
    }
    
    public function replica(): PDO
    {
        // Round-robin selection
        return $this->replicas[array_rand($this->replicas)];
    }
}

// Usage in repository
class UserRepository
{
    public function find(int $id): ?User
    {
        // Reads go to replica
        $stmt = $this->db->replica()->prepare(
            'SELECT * FROM users WHERE id = ?'
        );
        $stmt->execute([$id]);
        return $this->hydrate($stmt->fetch());
    }
    
    public function save(User $user): void
    {
        // Writes go to master
        $stmt = $this->db->master()->prepare(
            'INSERT INTO users (name, email) VALUES (?, ?)'
        );
        $stmt->execute([$user->name, $user->email]);
    }
}
```

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*