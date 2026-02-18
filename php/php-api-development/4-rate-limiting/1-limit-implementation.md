---
source_course: "php-api-development"
source_lesson: "php-api-development-rate-limit-implementation"
---

# Implementing Rate Limiting

Rate limiting prevents API abuse by restricting how many requests a client can make in a given time period.

## Token Bucket Algorithm

```php
<?php
class RateLimiter {
    public function __construct(
        private PDO $pdo,
        private int $maxRequests = 60,
        private int $windowSeconds = 60
    ) {}
    
    public function check(string $key): RateLimitResult {
        $now = time();
        $windowStart = $now - $this->windowSeconds;
        
        // Clean old records
        $stmt = $this->pdo->prepare(
            'DELETE FROM rate_limits WHERE `key` = :key AND timestamp < :window'
        );
        $stmt->execute(['key' => $key, 'window' => $windowStart]);
        
        // Count requests in window
        $stmt = $this->pdo->prepare(
            'SELECT COUNT(*) FROM rate_limits WHERE `key` = :key'
        );
        $stmt->execute(['key' => $key]);
        $count = (int) $stmt->fetchColumn();
        
        $remaining = max(0, $this->maxRequests - $count);
        $resetAt = $now + $this->windowSeconds;
        
        if ($count >= $this->maxRequests) {
            return new RateLimitResult(
                allowed: false,
                limit: $this->maxRequests,
                remaining: 0,
                resetAt: $resetAt
            );
        }
        
        // Record this request
        $stmt = $this->pdo->prepare(
            'INSERT INTO rate_limits (`key`, timestamp) VALUES (:key, :time)'
        );
        $stmt->execute(['key' => $key, 'time' => $now]);
        
        return new RateLimitResult(
            allowed: true,
            limit: $this->maxRequests,
            remaining: $remaining - 1,
            resetAt: $resetAt
        );
    }
}

class RateLimitResult {
    public function __construct(
        public bool $allowed,
        public int $limit,
        public int $remaining,
        public int $resetAt
    ) {}
    
    public function setHeaders(): void {
        header('X-RateLimit-Limit: ' . $this->limit);
        header('X-RateLimit-Remaining: ' . $this->remaining);
        header('X-RateLimit-Reset: ' . $this->resetAt);
        
        if (!$this->allowed) {
            header('Retry-After: ' . ($this->resetAt - time()));
        }
    }
}
```

## Rate Limit Middleware

```php
<?php
class RateLimitMiddleware {
    public function __construct(
        private RateLimiter $limiter
    ) {}
    
    public function handle(): void {
        // Use IP + user ID (if authenticated) as key
        $key = $this->getKey();
        
        $result = $this->limiter->check($key);
        $result->setHeaders();
        
        if (!$result->allowed) {
            http_response_code(429);
            header('Content-Type: application/json');
            echo json_encode([
                'error' => 'Too Many Requests',
                'message' => 'Rate limit exceeded. Try again later.',
                'retry_after' => $result->resetAt - time(),
            ]);
            exit;
        }
    }
    
    private function getKey(): string {
        $ip = $_SERVER['REMOTE_ADDR'] ?? 'unknown';
        
        // If authenticated, use user ID
        $userId = $GLOBALS['auth_user']['user_id'] ?? null;
        if ($userId) {
            return "user:$userId";
        }
        
        return "ip:$ip";
    }
}
```

## Different Limits for Different Endpoints

```php
<?php
class TieredRateLimiter {
    private array $tiers = [
        'default' => ['requests' => 60, 'window' => 60],
        'search' => ['requests' => 20, 'window' => 60],
        'write' => ['requests' => 10, 'window' => 60],
        'auth' => ['requests' => 5, 'window' => 300],
    ];
    
    public function check(string $key, string $tier = 'default'): RateLimitResult {
        $config = $this->tiers[$tier] ?? $this->tiers['default'];
        
        $limiter = new RateLimiter(
            $this->pdo,
            $config['requests'],
            $config['window']
        );
        
        return $limiter->check("$tier:$key");
    }
}

// Usage
$router->post('/auth/login', function() use ($rateLimiter) {
    $result = $rateLimiter->check($_SERVER['REMOTE_ADDR'], 'auth');
    // ...
});

$router->get('/search', function() use ($rateLimiter) {
    $result = $rateLimiter->check(getAuthUserId(), 'search');
    // ...
});
```

## Resources

- [Rate Limiting Strategies](https://cloud.google.com/architecture/rate-limiting-strategies-techniques) — Google Cloud rate limiting guide

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*