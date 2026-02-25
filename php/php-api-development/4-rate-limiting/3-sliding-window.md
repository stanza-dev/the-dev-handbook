---
source_course: "php-api-development"
source_lesson: "php-api-development-sliding-window"
---

# Sliding Window Rate Limiter

## Introduction
The fixed window algorithm has a well-known flaw: a client can make double the allowed requests by timing them at a window boundary. The sliding window algorithm solves this by smoothing the count across window boundaries. In this lesson you will learn how the sliding window counter works, build it in PHP, and understand when to choose it over the token bucket.

## Key Concepts
- **Fixed Window**: Divides time into discrete blocks (e.g., per minute) and counts requests within each block. The count resets completely at the boundary.
- **Sliding Window**: Combines the current window’s count with a weighted portion of the previous window’s count, creating a smooth transition between windows.
- **Window Weight**: The fraction of the previous window that still applies, calculated from how far into the current window we are.
- **Boundary Burst**: The problem where a client makes maximum requests at the end of one fixed window and the start of the next, effectively doubling their rate.

## Real World Context
Cloudflare and Nginx both use sliding window rate limiting in production. It provides a better approximation of a true sliding window (which would require storing every request timestamp) while using only slightly more memory than a fixed window. If your API needs accurate rate limiting without the complexity of a token bucket, the sliding window counter is an excellent choice.

## Deep Dive

### The Fixed Window Boundary Problem

Consider a limit of 100 requests per minute with a fixed window:

```php
<?php
// Fixed window vulnerability:
// Window 1 (12:00:00 - 12:00:59): Client sends 0 requests until 12:00:50
// At 12:00:50: Client sends 100 requests (allowed — within limit)
// At 12:01:01: New window starts, counter resets to 0
// At 12:01:01: Client sends 100 requests (allowed — within limit)
//
// Result: 200 requests in ~11 seconds, despite a 100/minute limit
```

The client exploited the boundary to effectively double the rate. The sliding window eliminates this loophole.

### How the Sliding Window Counter Works

The algorithm uses two fixed windows (current and previous) but weights the previous window’s count based on the overlap:

```php
<?php
// Sliding window formula:
// effectiveCount = currentWindowCount + (previousWindowCount * overlapWeight)
// overlapWeight = 1 - (elapsedInCurrentWindow / windowSize)
//
// Example: 60-second windows, we are 15 seconds into the current window
// overlapWeight = 1 - (15 / 60) = 0.75
// If previous window had 80 requests and current has 30:
// effectiveCount = 30 + (80 * 0.75) = 30 + 60 = 90
```

The further into the current window we are, the less the previous window counts. At the exact boundary, the weight is 1.0 (full overlap). At the end of the current window, the weight approaches 0.0.

### PHP Implementation

Here is a complete sliding window rate limiter:

```php
<?php
class SlidingWindowRateLimiter {
    public function __construct(
        private readonly int $maxRequests, // Max requests per window
        private readonly int $windowSize,  // Window size in seconds
    ) {}

    public function isAllowed(string $clientId): bool {
        $now = time();
        $currentWindow = (int) floor($now / $this->windowSize);
        $previousWindow = $currentWindow - 1;

        $currentKey = "sw:{$clientId}:{$currentWindow}";
        $previousKey = "sw:{$clientId}:{$previousWindow}";

        // Get counts for both windows
        $currentCount = (int) (apcu_fetch($currentKey) ?: 0);
        $previousCount = (int) (apcu_fetch($previousKey) ?: 0);

        // Calculate the overlap weight
        $elapsedInWindow = $now - ($currentWindow * $this->windowSize);
        $overlapWeight = 1 - ($elapsedInWindow / $this->windowSize);

        // Weighted count
        $effectiveCount = $currentCount + ($previousCount * $overlapWeight);

        if ($effectiveCount >= $this->maxRequests) {
            return false; // Rate limited
        }

        // Increment the current window count
        $newCount = apcu_inc($currentKey);
        if ($newCount === 1 || $newCount === false) {
            apcu_store($currentKey, 1, $this->windowSize * 2);
        }

        return true;
    }

    public function remaining(string $clientId): int {
        $now = time();
        $currentWindow = (int) floor($now / $this->windowSize);
        $previousWindow = $currentWindow - 1;

        $currentCount = (int) (apcu_fetch("sw:{$clientId}:{$currentWindow}") ?: 0);
        $previousCount = (int) (apcu_fetch("sw:{$clientId}:{$previousWindow}") ?: 0);

        $elapsedInWindow = $now - ($currentWindow * $this->windowSize);
        $overlapWeight = 1 - ($elapsedInWindow / $this->windowSize);
        $effectiveCount = $currentCount + ($previousCount * $overlapWeight);

        return max(0, $this->maxRequests - (int) ceil($effectiveCount));
    }
}

// Usage: 100 requests per 60-second window
$limiter = new SlidingWindowRateLimiter(maxRequests: 100, windowSize: 60);

if (!$limiter->isAllowed($clientId)) {
    http_response_code(429);
    header('Retry-After: ' . (60 - (time() % 60)));
    echo json_encode(['error' => 'Rate limit exceeded']);
    exit;
}
```

The implementation stores two counters (current and previous window) and computes the weighted total on each request. The TTL is set to twice the window size to ensure the previous window’s counter is available for the full duration of the current window.

### Sliding Window vs Token Bucket

Both algorithms are production-quality. Here is when to choose each:

```php
<?php
// When to use Sliding Window:
// - You want a predictable, fixed request-per-window limit
// - Your clients expect a simple "X requests per minute" contract
// - You need minimal state (just two counters per client)
//
// When to use Token Bucket:
// - You want to allow controlled bursts above the average rate
// - Your clients are bursty (e.g., page loads trigger multiple API calls)
// - You need fine-grained control over burst size vs sustained rate
//
// Memory comparison (per client):
// Fixed window:   1 counter + 1 timestamp  = ~16 bytes
// Sliding window:  2 counters + 1 timestamp = ~24 bytes
// Token bucket:    1 float + 1 timestamp    = ~16 bytes
```

The sliding window provides more accurate rate limiting than the fixed window at negligible extra cost. The token bucket is better when you want to decouple burst size from average rate.

## Common Pitfalls
1. **Forgetting to keep the previous window’s counter** — If the TTL on the previous window’s key is too short, the counter disappears before the current window is over, and the sliding calculation falls back to a fixed window. Set the TTL to at least twice the window size.
2. **Using floating-point comparison for the limit check** — The effective count is a float. Use `>=` rather than `>` to avoid off-by-one errors where 99.5 rounds down and lets an extra request through.

## Best Practices
1. **Round the effective count up when reporting remaining** — Use `ceil()` when calculating the remaining count to report. This prevents telling a client they have 1 request left when they actually have 0.7.
2. **Store both windows under predictable keys** — Use a key format like `sw:{clientId}:{windowNumber}` where the window number is `floor(time / windowSize)`. This makes debugging easy: you can look up any client’s counters by their ID and the current timestamp.

## Summary
- The sliding window algorithm fixes the fixed window’s boundary burst vulnerability by weighting the previous window’s count.
- The formula is: `effectiveCount = current + (previous * overlapWeight)`, where `overlapWeight` decreases from 1.0 to 0.0 over the current window.
- It requires only two counters per client, making it nearly as memory-efficient as a fixed window.
- Choose sliding window for simple request-per-window contracts; choose token bucket when you need burst control.
- Always set TTLs to at least twice the window size so the previous counter survives.

## Code Examples

**Sliding window calculation showing how the previous window's count is weighted by overlap to prevent boundary burst exploits**

```php
<?php
// Sliding window rate limit calculation
$windowSize = 60; // seconds
$now = time();
$currentWindow = (int) floor($now / $windowSize);
$previousWindow = $currentWindow - 1;

// Fetch counters
$currentCount = 30;   // 30 requests in current window
$previousCount = 80;  // 80 requests in previous window

// Calculate overlap weight (how much of the previous window counts)
$elapsedInWindow = $now - ($currentWindow * $windowSize); // e.g., 15 seconds
$overlapWeight = 1 - ($elapsedInWindow / $windowSize);     // e.g., 0.75

// Effective count: current + weighted previous
$effectiveCount = $currentCount + ($previousCount * $overlapWeight);
// 30 + (80 * 0.75) = 90 — under a limit of 100, so allowed
```


## Resources

- [PHP floor function](https://www.php.net/manual/en/function.floor.php) — Used to calculate the current window number from a timestamp
- [PHP apcu_inc function](https://www.php.net/manual/en/function.apcu-inc.php) — Atomic increment for APCu cache counters used in rate limiting

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*