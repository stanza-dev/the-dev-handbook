---
source_course: "php-security"
source_lesson: "php-security-rate-limiting"
---

# Rate Limiting

## Introduction
Rate limiting prevents abuse by restricting how often actions can be performed.

## Key Concepts
- **Token Bucket Algorithm**: A rate limiting strategy where tokens are added at a fixed rate and consumed per request.
- **Sliding Window**: Tracks request counts in overlapping time windows for smoother rate limit enforcement.
- **HTTP 429 Too Many Requests**: The standard status code for rate-limited responses, with `Retry-After` header.
- **Distributed Rate Limiting**: Using Redis or Memcached to enforce rate limits across multiple application servers.

## Real World Context
Rate limiting is critical for preventing brute-force login attacks, API abuse, and denial-of-service. GitHub's API limits unauthenticated requests to 60/hour and authenticated to 5,000/hour. Without rate limiting, a single attacker can overwhelm your login endpoint with credential stuffing attacks.

## Deep Dive
### Intro

Rate limiting prevents abuse by restricting how often actions can be performed.

### Simple rate limiter

```php
<?php
class RateLimiter {
    public function __construct(
        private PDO $pdo
    ) {}
    
    public function attempt(
        string $key,
        int $maxAttempts,
        int $windowSeconds
    ): bool {
        $now = time();
        $windowStart = $now - $windowSeconds;
        
        // Clean old entries
        $stmt = $this->pdo->prepare(
            'DELETE FROM rate_limits WHERE key = :key AND timestamp < :window'
        );
        $stmt->execute(['key' => $key, 'window' => $windowStart]);
        
        // Count recent attempts
        $stmt = $this->pdo->prepare(
            'SELECT COUNT(*) FROM rate_limits WHERE key = :key'
        );
        $stmt->execute(['key' => $key]);
        $count = $stmt->fetchColumn();
        
        if ($count >= $maxAttempts) {
            return false;  // Rate limited
        }
        
        // Record this attempt
        $stmt = $this->pdo->prepare(
            'INSERT INTO rate_limits (key, timestamp) VALUES (:key, :time)'
        );
        $stmt->execute(['key' => $key, 'time' => $now]);
        
        return true;
    }
}

// Usage
$limiter = new RateLimiter($pdo);
$key = 'login:' . $_SERVER['REMOTE_ADDR'];

if (!$limiter->attempt($key, maxAttempts: 5, windowSeconds: 300)) {
    http_response_code(429);
    die('Too many requests. Please wait 5 minutes.');
}
```

### Token bucket with redis

```php
<?php
class TokenBucketLimiter {
    public function __construct(
        private Redis $redis
    ) {}
    
    public function consume(
        string $key,
        int $capacity,
        int $refillRate,
        int $refillInterval = 1
    ): bool {
        $now = microtime(true);
        $bucketKey = "bucket:$key";
        
        $bucket = $this->redis->hGetAll($bucketKey);
        
        if (empty($bucket)) {
            $tokens = $capacity - 1;
            $lastRefill = $now;
        } else {
            $tokens = (float) $bucket['tokens'];
            $lastRefill = (float) $bucket['last_refill'];
            
            // Refill tokens based on time elapsed
            $elapsed = $now - $lastRefill;
            $refillAmount = floor($elapsed / $refillInterval) * $refillRate;
            $tokens = min($capacity, $tokens + $refillAmount);
            $lastRefill = $now;
            
            if ($tokens < 1) {
                return false;  // No tokens available
            }
            
            $tokens--;
        }
        
        $this->redis->hMSet($bucketKey, [
            'tokens' => $tokens,
            'last_refill' => $lastRefill,
        ]);
        $this->redis->expire($bucketKey, 3600);
        
        return true;
    }
}
```

## Common Pitfalls
1. **Rate limiting by IP only** — Shared IPs (corporate NAT, mobile carriers) can affect legitimate users. Combine IP-based limits with user/API key-based limits.
2. **Not including Retry-After headers** — Well-behaved clients need to know when they can retry. Always include the `Retry-After` header with 429 responses.

## Best Practices
1. **Use Redis for distributed rate limiting** — The `INCR` + `EXPIRE` pattern in Redis provides atomic, cross-server rate limiting with minimal overhead.
2. **Apply tiered rate limits** — Use different limits for different endpoints: stricter for login/password-reset, more generous for read-only API endpoints.

## Summary
- Rate limiting prevents brute-force attacks, credential stuffing, and API abuse.
- Return HTTP 429 with a `Retry-After` header when limits are exceeded.
- Use Redis-backed rate limiting for distributed applications with multiple servers.

## Code Examples

**Production-ready sliding window rate limiter**

```php
<?php
declare(strict_types=1);

// Production rate limiter with different strategies
interface RateLimiterInterface {
    public function attempt(string $key): bool;
    public function getRemainingAttempts(string $key): int;
    public function getRetryAfter(string $key): int;
}

class SlidingWindowLimiter implements RateLimiterInterface {
    public function __construct(
        private PDO $pdo,
        private int $maxAttempts = 60,
        private int $windowSeconds = 60
    ) {}
    
    public function attempt(string $key): bool {
        $now = time();
        $windowStart = $now - $this->windowSeconds;
        
        // Atomic operation with transaction
        $this->pdo->beginTransaction();
        
        try {
            // Remove expired entries
            $stmt = $this->pdo->prepare(
                'DELETE FROM rate_limits WHERE `key` = :key AND created_at < :window'
            );
            $stmt->execute(['key' => $key, 'window' => $windowStart]);
            
            // Count current window
            $stmt = $this->pdo->prepare(
                'SELECT COUNT(*) FROM rate_limits WHERE `key` = :key'
            );
            $stmt->execute(['key' => $key]);
            $count = (int) $stmt->fetchColumn();
            
            if ($count >= $this->maxAttempts) {
                $this->pdo->commit();
                return false;
            }
            
            // Record attempt
            $stmt = $this->pdo->prepare(
                'INSERT INTO rate_limits (`key`, created_at) VALUES (:key, :time)'
            );
            $stmt->execute(['key' => $key, 'time' => $now]);
            
            $this->pdo->commit();
            return true;
            
        } catch (Exception $e) {
            $this->pdo->rollBack();
            throw $e;
        }
    }
    
    public function getRemainingAttempts(string $key): int {
        $windowStart = time() - $this->windowSeconds;
        
        $stmt = $this->pdo->prepare(
            'SELECT COUNT(*) FROM rate_limits WHERE `key` = :key AND created_at >= :window'
        );
        $stmt->execute(['key' => $key, 'window' => $windowStart]);
        $count = (int) $stmt->fetchColumn();
        
        return max(0, $this->maxAttempts - $count);
    }
    
    public function getRetryAfter(string $key): int {
        $stmt = $this->pdo->prepare(
            'SELECT MIN(created_at) FROM rate_limits WHERE `key` = :key'
        );
        $stmt->execute(['key' => $key]);
        $oldest = $stmt->fetchColumn();
        
        if (!$oldest) {
            return 0;
        }
        
        return max(0, ($oldest + $this->windowSeconds) - time());
    }
}

// Middleware usage
function rateLimitMiddleware(RateLimiterInterface $limiter): void {
    $key = 'api:' . ($_SERVER['REMOTE_ADDR'] ?? 'unknown');
    
    if (!$limiter->attempt($key)) {
        $retryAfter = $limiter->getRetryAfter($key);
        header('Retry-After: ' . $retryAfter);
        header('X-RateLimit-Remaining: 0');
        http_response_code(429);
        die(json_encode(['error' => 'Too Many Requests']));
    }
    
    header('X-RateLimit-Remaining: ' . $limiter->getRemainingAttempts($key));
}
?>
```


## Resources

- [Rate Limiting Best Practices](https://github.com/phpredis/phpredis) — PHP Redis extension for implementing rate limiters

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*