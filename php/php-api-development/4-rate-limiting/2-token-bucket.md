---
source_course: "php-api-development"
source_lesson: "php-api-development-token-bucket"
---

# Token Bucket Algorithm

## Introduction
The token bucket is the most popular rate limiting algorithm for APIs because it enforces an average request rate while allowing short bursts of traffic. Think of it as a bucket that fills with tokens at a steady rate: each request takes one token, and when the bucket is empty, requests are rejected until more tokens arrive. In this lesson you will understand the mechanics of the token bucket and build a working implementation in PHP.

## Key Concepts
- **Token**: A virtual unit that represents permission to make one API request.
- **Bucket Capacity**: The maximum number of tokens the bucket can hold. This determines the maximum burst size.
- **Refill Rate**: How many tokens are added to the bucket per unit of time (e.g., 10 tokens per second).
- **Burst**: A rapid sequence of requests that temporarily exceeds the average rate. The token bucket allows bursts up to the bucket capacity.

## Real World Context
Amazon API Gateway, Stripe, and most CDN providers use the token bucket algorithm for rate limiting. It is the best fit for web APIs because real clients are bursty: a page load might trigger 5 API calls in 100ms, then nothing for 10 seconds. A fixed window would either reject those 5 fast calls or allow too many total. The token bucket handles both scenarios naturally.

## Deep Dive

### How the Token Bucket Works

The algorithm follows four simple rules:

1. The bucket starts full (capacity = maximum burst size).
2. Each request removes one token from the bucket.
3. Tokens are added at a constant rate (the refill rate).
4. If the bucket is empty when a request arrives, the request is rejected.

Tokens that would exceed the bucket capacity are discarded — the bucket never overflows.

Here is a visual representation of the flow:

```php
<?php
// Conceptual visualization (not runnable)
// Bucket capacity: 5 tokens, Refill: 1 token/second
//
// Time 0s: [*][*][*][*][*]  — 5 tokens (full)
// Request!: [*][*][*][*][ ]  — 4 tokens
// Request!: [*][*][*][ ][ ]  — 3 tokens
// Request!: [*][*][ ][ ][ ]  — 2 tokens
// Time 1s:  [*][*][*][ ][ ]  — 3 tokens (1 refilled)
// Request!: [*][*][ ][ ][ ]  — 2 tokens
// Time 2s:  [*][*][*][ ][ ]  — 3 tokens (1 refilled)
```

The bucket allows up to 5 requests in a burst, but the long-term average is limited to 1 request per second.

### PHP Implementation with APCu

APCu is a local in-memory cache that ships as a PHP extension. It is the simplest backend for single-server rate limiting. For multi-server deployments, you would swap APCu for Redis.

Here is a complete token bucket implementation:

```php
<?php
class TokenBucket {
    public function __construct(
        private readonly int $capacity,    // Max tokens (burst size)
        private readonly float $refillRate, // Tokens added per second
    ) {}

    /**
     * Attempt to consume a token for the given client.
     * Returns true if allowed, false if rate limited.
     */
    public function consume(string $clientId): bool {
        $key = 'bucket:' . $clientId;
        $now = microtime(true);

        // Fetch the current bucket state
        $bucket = apcu_fetch($key);

        if ($bucket === false) {
            // First request — initialize a full bucket minus one token
            apcu_store($key, [
                'tokens'    => $this->capacity - 1,
                'lastRefill' => $now,
            ], 3600); // TTL: 1 hour
            return true;
        }

        // Calculate tokens to add since last refill
        $elapsed = $now - $bucket['lastRefill'];
        $newTokens = $elapsed * $this->refillRate;
        $tokens = min($this->capacity, $bucket['tokens'] + $newTokens);

        if ($tokens < 1) {
            // Bucket is empty — reject the request
            return false;
        }

        // Consume one token
        apcu_store($key, [
            'tokens'    => $tokens - 1,
            'lastRefill' => $now,
        ], 3600);

        return true;
    }

    /**
     * Get the number of tokens remaining for a client.
     */
    public function remaining(string $clientId): int {
        $bucket = apcu_fetch('bucket:' . $clientId);

        if ($bucket === false) {
            return $this->capacity;
        }

        $elapsed = microtime(true) - $bucket['lastRefill'];
        $tokens = min($this->capacity, $bucket['tokens'] + ($elapsed * $this->refillRate));

        return (int) floor($tokens);
    }
}
```

Each call to `consume()` recalculates the token count based on elapsed time, so there is no background process adding tokens — refills happen lazily on each request.

### Using the Token Bucket in Middleware

Here is how to wire the token bucket into your API as middleware:

```php
<?php
class RateLimitMiddleware {
    private TokenBucket $bucket;

    public function __construct(int $capacity, float $refillRate) {
        $this->bucket = new TokenBucket($capacity, $refillRate);
    }

    public function __invoke(Request $request, callable $next): array {
        $clientId = $this->getClientId($request);

        if (!$this->bucket->consume($clientId)) {
            $retryAfter = (int) ceil(1 / $this->bucket->remaining($clientId) ?: 1);

            http_response_code(429);
            header('Retry-After: ' . $retryAfter);
            header('X-RateLimit-Limit: ' . $this->bucket->remaining($clientId));
            header('X-RateLimit-Remaining: 0');

            return ['error' => 'Too Many Requests'];
        }

        $remaining = $this->bucket->remaining($clientId);
        header('X-RateLimit-Remaining: ' . $remaining);

        return $next($request);
    }

    private function getClientId(Request $request): string {
        $user = $request->getAuthUser();
        return $user ? 'user:' . $user->sub : 'ip:' . $request->ip();
    }
}

// Usage: allow 100 requests per minute (bucket=100, refill=1.67/sec)
$rateLimiter = new RateLimitMiddleware(100, 100 / 60);
$router->group('/api', function ($group) { /* routes */ })->middleware($rateLimiter);
```

The middleware checks the bucket before every request. If the bucket is empty, it returns a 429 with a `Retry-After` header. Otherwise, it passes the request through and reports the remaining tokens.

### Redis Backend for Multi-Server Deployments

For APIs running on multiple servers, you need a shared token store. Redis is the standard choice:

```php
<?php
class RedisTokenBucket {
    public function __construct(
        private readonly \Redis $redis,
        private readonly int $capacity,
        private readonly float $refillRate,
    ) {}

    public function consume(string $clientId): bool {
        $key = 'bucket:' . $clientId;
        $now = microtime(true);

        // Use a Lua script for atomic read-modify-write
        $script = <<<'LUA'
        local key = KEYS[1]
        local capacity = tonumber(ARGV[1])
        local refillRate = tonumber(ARGV[2])
        local now = tonumber(ARGV[3])

        local data = redis.call('HMGET', key, 'tokens', 'lastRefill')
        local tokens = tonumber(data[1]) or capacity
        local lastRefill = tonumber(data[2]) or now

        local elapsed = now - lastRefill
        tokens = math.min(capacity, tokens + elapsed * refillRate)

        if tokens < 1 then
            return 0
        end

        redis.call('HMSET', key, 'tokens', tokens - 1, 'lastRefill', now)
        redis.call('EXPIRE', key, 3600)
        return 1
        LUA;

        return (bool) $this->redis->eval($script, [$key, $this->capacity, $this->refillRate, $now], 1);
    }
}
```

The Lua script runs atomically inside Redis, preventing race conditions when multiple servers handle requests for the same client simultaneously.

## Common Pitfalls
1. **Not handling concurrent requests atomically** — If two requests check the bucket at the same time, both might see 1 remaining token and both consume it, exceeding the limit. Use Redis Lua scripts or APCu’s atomic increment to prevent this.
2. **Setting the bucket capacity too low** — A capacity of 1 with a refill rate of 1/second means the client can never make two requests back-to-back, even if they average under the limit. Set the capacity to allow reasonable bursts (e.g., 10-20x the per-second rate).

## Best Practices
1. **Use lazy refill calculation** — Do not run a background process to add tokens. Calculate the refill on each request based on elapsed time. This is simpler, uses no extra resources, and scales to millions of buckets.
2. **Set a TTL on bucket keys** — Inactive clients should not consume memory forever. Set a TTL (e.g., 1 hour) on bucket records so they are automatically cleaned up.

## Summary
- The token bucket algorithm allows short bursts while enforcing a long-term average rate.
- Each request consumes one token; tokens refill at a constant rate up to the bucket capacity.
- Use APCu for single-server deployments and Redis with Lua scripts for multi-server setups.
- Lazy refill calculation (computing tokens on each request) is simpler and more efficient than background token addition.
- Set a TTL on bucket keys to prevent unbounded memory growth from inactive clients.

## Code Examples

**Token bucket implementation using APCu for lazy refill calculation — tokens are computed on each request based on elapsed time**

```php
<?php
class TokenBucket {
    public function __construct(
        private readonly int $capacity,     // Max burst size
        private readonly float $refillRate, // Tokens per second
    ) {}

    public function consume(string $clientId): bool {
        $key = 'bucket:' . $clientId;
        $now = microtime(true);
        $bucket = apcu_fetch($key);

        if ($bucket === false) {
            apcu_store($key, ['tokens' => $this->capacity - 1, 'lastRefill' => $now], 3600);
            return true; // First request allowed
        }

        $elapsed = $now - $bucket['lastRefill'];
        $tokens = min($this->capacity, $bucket['tokens'] + $elapsed * $this->refillRate);

        if ($tokens < 1) {
            return false; // Rate limited
        }

        apcu_store($key, ['tokens' => $tokens - 1, 'lastRefill' => $now], 3600);
        return true; // Request allowed
    }
}

// Allow 100 req/min with burst of 100
$bucket = new TokenBucket(capacity: 100, refillRate: 100 / 60);
```


## Resources

- [PHP APCu extension](https://www.php.net/manual/en/book.apcu.php) — PHP documentation for the APCu user cache used as a token bucket backend
- [PHP microtime function](https://www.php.net/manual/en/function.microtime.php) — High-precision timestamp function used for accurate token refill calculations

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*