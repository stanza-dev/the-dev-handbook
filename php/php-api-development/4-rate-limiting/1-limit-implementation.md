---
source_course: "php-api-development"
source_lesson: "php-api-development-rate-limit-implementation"
---

# Why Rate Limiting Matters

## Introduction
Without rate limiting, a single misbehaving client can overwhelm your API, driving up infrastructure costs, degrading performance for everyone, and potentially taking your service offline. Rate limiting is the practice of controlling how many requests a client can make in a given time window. In this lesson you will learn why rate limiting is essential and survey the most common algorithms used to implement it.

## Key Concepts
- **Rate Limiting**: Restricting the number of API requests a client can make within a defined time period.
- **Throttling**: Slowing down or queuing requests instead of rejecting them outright. Often used interchangeably with rate limiting, but technically a softer approach.
- **DDoS (Distributed Denial of Service)**: An attack where many clients flood a server with requests to make it unavailable. Rate limiting is the first line of defense.
- **Fair Usage**: Ensuring that no single client consumes a disproportionate share of server resources.
- **429 Too Many Requests**: The HTTP status code returned when a client exceeds the rate limit.

## Real World Context
Every public API enforces rate limits. GitHub allows 5,000 authenticated requests per hour. Stripe limits to 100 requests per second. Twitter caps at 900 reads per 15-minute window. Without these limits, a bug in a single client’s retry loop could cost thousands of dollars in compute and storage. As a PHP developer building APIs, you will implement rate limiting to protect your infrastructure, control costs, and provide fair access to all clients.

## Deep Dive

### The Four Reasons to Rate Limit

Rate limiting serves four distinct purposes, each important in different contexts:

**1. DDoS Protection** — Without rate limiting, a malicious actor can send millions of requests to exhaust your server’s CPU, memory, and database connections. Even a basic rate limiter can block the majority of attack traffic.

**2. Fair Usage** — In a multi-tenant API, one aggressive client can starve others of resources. Per-client rate limits ensure equitable access.

**3. Cost Control** — Cloud infrastructure charges per request, per compute second, and per database query. Unbounded traffic from a misbehaving client can run up unexpected bills.

**4. Stability** — Databases and downstream services have throughput limits. Rate limiting acts as a pressure valve, keeping your entire system within safe operating parameters.

Here is what a rate-limited response looks like in practice:

```php
<?php
// When the client exceeds the limit, respond with 429
http_response_code(429);
header('Retry-After: 60'); // Tell the client when to try again
header('X-RateLimit-Limit: 100');
header('X-RateLimit-Remaining: 0');
header('X-RateLimit-Reset: ' . (time() + 60));

echo json_encode([
    'error'   => 'Too Many Requests',
    'message' => 'Rate limit exceeded. Try again in 60 seconds.',
]);
```

The response includes standard headers so the client knows its limit, how many requests remain, and when the limit resets. Good API design communicates limits clearly.

### Common Rate Limiting Algorithms

There are four main algorithms, each with different trade-offs:

**Fixed Window** — Counts requests in fixed time blocks (e.g., per minute). Simple but allows bursts at window boundaries.

```php
<?php
// Fixed window: count requests per minute
$windowKey = 'rate:' . $clientId . ':' . date('Y-m-d-H-i');
$count = $cache->increment($windowKey);

if ($count === 1) {
    $cache->expire($windowKey, 60); // TTL: 1 minute
}

if ($count > $maxRequests) {
    // Reject the request
    http_response_code(429);
    echo json_encode(['error' => 'Rate limit exceeded']);
    exit;
}
```

The fixed window approach is easy to implement but has a flaw: a client can make `$maxRequests` at the end of one window and `$maxRequests` at the start of the next, effectively doubling their rate for a brief period.

**Sliding Window** — Smooths out the fixed window boundary problem by weighting the previous window’s count. More accurate but slightly more complex.

**Token Bucket** — Each client has a "bucket" that fills with tokens at a steady rate. Each request consumes a token. If the bucket is empty, the request is rejected. This allows controlled bursts while enforcing an average rate.

**Leaky Bucket** — Requests enter a queue (bucket) and are processed at a fixed rate. Excess requests overflow and are rejected. This produces the smoothest output rate.

Here is a comparison to help you choose:

```php
<?php
// Algorithm comparison (not runnable — conceptual overview)
$algorithms = [
    'Fixed Window'   => ['complexity' => 'Low',    'burst_handling' => 'Poor',  'memory' => 'Low'],
    'Sliding Window'  => ['complexity' => 'Medium', 'burst_handling' => 'Good',  'memory' => 'Medium'],
    'Token Bucket'    => ['complexity' => 'Medium', 'burst_handling' => 'Best',  'memory' => 'Low'],
    'Leaky Bucket'    => ['complexity' => 'Medium', 'burst_handling' => 'N/A',   'memory' => 'Medium'],
];
// Token Bucket is the most popular choice for API rate limiting
```

The token bucket algorithm is the most widely used in production APIs because it allows short bursts (good for bursty client behavior) while enforcing a long-term average rate.

### Identifying Clients

Rate limiting requires identifying who is making the request. Common strategies include:

```php
<?php
function getClientIdentifier(Request $request): string {
    // Option 1: API key (best for authenticated APIs)
    $apiKey = $request->header('X-API-Key');
    if ($apiKey !== null) {
        return 'key:' . hash('sha256', $apiKey);
    }

    // Option 2: Authenticated user ID
    $user = $request->getAuthUser();
    if ($user !== null) {
        return 'user:' . $user->sub;
    }

    // Option 3: IP address (fallback for unauthenticated requests)
    return 'ip:' . $request->ip();
}
```

Using the API key or user ID is more reliable than IP address, because multiple users can share an IP (corporate networks, VPNs) or a single user can rotate IPs.

## Common Pitfalls
1. **Rate limiting by IP only** — Users behind corporate proxies or VPNs share a single IP address. Rate limiting by IP alone punishes all users behind that proxy. Prefer API key or user ID when available.
2. **Not sending rate limit headers** — If you reject requests with a plain 429 and no headers, clients have no way to know their limit or when to retry. Always include `X-RateLimit-Limit`, `X-RateLimit-Remaining`, and `Retry-After`.

## Best Practices
1. **Choose the right algorithm for your traffic pattern** — Use token bucket for APIs with bursty traffic (most common). Use fixed window for simple internal services where burst accuracy is not critical.
2. **Apply different limits to different tiers** — Free-tier clients get 100 requests/minute, paid clients get 1,000, and internal services get 10,000. This aligns rate limiting with your business model and protects shared resources.

## Summary
- Rate limiting protects against DDoS attacks, ensures fair usage, controls costs, and maintains system stability.
- The four main algorithms are fixed window, sliding window, token bucket, and leaky bucket.
- The 429 status code signals rate limit exceeded; always include `Retry-After` and `X-RateLimit-*` headers.
- Identify clients by API key or user ID when possible, falling back to IP address for anonymous requests.
- Token bucket is the most popular algorithm for production APIs because it handles bursts gracefully.

## Code Examples

**A fixed-window rate limiter that tracks requests per minute and returns standard rate limit headers with every response**

```php
<?php
// Basic rate limit check and 429 response
$clientId = getClientIdentifier($request);
$windowKey = 'rate:' . $clientId . ':' . date('Y-m-d-H-i');
$requestCount = $cache->increment($windowKey);

if ($requestCount === 1) {
    $cache->expire($windowKey, 60);
}

$limit = 100;
$remaining = max(0, $limit - $requestCount);

header("X-RateLimit-Limit: $limit");
header("X-RateLimit-Remaining: $remaining");

if ($requestCount > $limit) {
    http_response_code(429);
    header('Retry-After: 60');
    echo json_encode(['error' => 'Rate limit exceeded']);
    exit;
}

// Request allowed — continue processing
```


## Resources

- [PHP time function](https://www.php.net/manual/en/function.time.php) — PHP Unix timestamp function used for window calculations and rate limit resets
- [PHP header function](https://www.php.net/manual/en/function.header.php) — Setting HTTP response headers including rate limit headers

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*