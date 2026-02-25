---
source_course: "php-api-development"
source_lesson: "php-api-development-rate-limit-headers"
---

# Rate Limiting Headers and Client Communication

## Introduction
Rate limiting is only half the job. The other half is telling clients about their limits so they can adjust their behavior proactively. Without clear communication, clients waste time on requests that will be rejected, and developers waste time debugging mysterious 429 errors. In this lesson you will learn the standard rate limiting headers, how to implement them in PHP, and how PHP 8.5’s `#[\NoDiscard]` attribute helps prevent silent bugs in rate limit check results.

## Key Concepts
- **X-RateLimit-Limit**: Header that tells the client the maximum number of requests allowed in the current window.
- **X-RateLimit-Remaining**: Header that tells the client how many requests they have left in the current window.
- **X-RateLimit-Reset**: Header that tells the client when the rate limit window resets (as a Unix timestamp).
- **Retry-After**: Standard HTTP header (RFC 7231) that tells the client how many seconds to wait before retrying after a 429 response.
- **#[\NoDiscard]**: A PHP 8.5 attribute that triggers a warning if a function’s return value is ignored, preventing bugs where developers forget to check the rate limit result.

## Real World Context
Well-designed APIs like GitHub, Stripe, and Twitter include rate limit headers on every response, not just 429s. This lets client developers build adaptive behavior: slow down when remaining is low, back off when they hit the limit, and resume at the right time. If your API returns bare 429s without headers, expect a flood of support tickets from confused integrators.

## Deep Dive

### The Standard Rate Limit Headers

While not part of an official RFC (a draft exists as RateLimit Fields for HTTP), the following headers are a de facto standard used by most major APIs:

Here is how they appear in a response:

```php
<?php
// Headers on a successful response (200 OK)
// X-RateLimit-Limit: 100
// X-RateLimit-Remaining: 73
// X-RateLimit-Reset: 1709049600

// Headers on a rate-limited response (429 Too Many Requests)
// X-RateLimit-Limit: 100
// X-RateLimit-Remaining: 0
// X-RateLimit-Reset: 1709049600
// Retry-After: 42
```

The key insight is that `X-RateLimit-*` headers appear on **every** response, not just 429s. `Retry-After` only appears on 429 responses.

### Implementing Headers in PHP

Here is a complete rate limit response helper:

```php
<?php
class RateLimitResponse {
    public function __construct(
        private readonly int $limit,
        private readonly int $remaining,
        private readonly int $resetTimestamp,
    ) {}

    /**
     * Send rate limit headers. Call this on EVERY response.
     */
    public function sendHeaders(): void {
        header('X-RateLimit-Limit: ' . $this->limit);
        header('X-RateLimit-Remaining: ' . $this->remaining);
        header('X-RateLimit-Reset: ' . $this->resetTimestamp);
    }

    /**
     * Send a 429 response with Retry-After.
     */
    public function sendRateLimited(): never {
        $retryAfter = max(1, $this->resetTimestamp - time());

        http_response_code(429);
        $this->sendHeaders();
        header('Retry-After: ' . $retryAfter);

        echo json_encode([
            'error'       => 'Too Many Requests',
            'retry_after' => $retryAfter,
            'limit'       => $this->limit,
        ]);
        exit;
    }
}

// Usage in middleware
$remaining = $rateLimiter->remaining($clientId);
$resetTime = $rateLimiter->resetTime($clientId);
$response = new RateLimitResponse(limit: 100, remaining: $remaining, resetTimestamp: $resetTime);

if ($remaining <= 0) {
    $response->sendRateLimited(); // Sends 429 + headers + exits
}

$response->sendHeaders(); // Sends headers on normal responses too
```

Using the `never` return type on `sendRateLimited()` tells PHP and static analyzers that this method always terminates execution, preventing unreachable code warnings.

### PHP 8.5’s #[\NoDiscard] for Rate Limit Checks

A common bug is calling the rate limit check method but forgetting to use the result:

```php
<?php
// BUG: the return value is ignored — every request passes through!
$rateLimiter->isAllowed($clientId);  // Returns false, but nobody checks it
processRequest($request);            // Runs anyway
```

PHP 8.5’s `#[\NoDiscard]` attribute makes the compiler emit a warning when a function’s return value is discarded:

```php
<?php
// PHP 8.5: #[\NoDiscard] prevents ignored return values
class RateLimiter {
    #[\NoDiscard('Rate limit check result must be used to enforce the limit')]
    public function isAllowed(string $clientId): bool {
        $remaining = $this->bucket->remaining($clientId);
        return $remaining > 0;
    }

    #[\NoDiscard('Consume result indicates if the request was allowed')]
    public function consume(string $clientId): bool {
        return $this->bucket->consume($clientId);
    }
}

// Now this triggers a warning:
$rateLimiter->isAllowed($clientId);  // Warning: return value of isAllowed() is discarded

// The correct usage:
if (!$rateLimiter->isAllowed($clientId)) {
    // Handle rate limiting
}
```

By annotating rate limit methods with `#[\NoDiscard]`, you make the API self-documenting and prevent the entire class of "forgot to check the result" bugs at compile time.

### Putting It All Together: Complete Middleware

Here is a production-ready rate limit middleware that sends proper headers on every response:

```php
<?php
class RateLimitMiddleware {
    public function __construct(
        private readonly TokenBucket $bucket,
        private readonly int $limit,
    ) {}

    public function __invoke(Request $request, callable $next): array {
        $clientId = $this->getClientId($request);
        $allowed = $this->bucket->consume($clientId);
        $remaining = $this->bucket->remaining($clientId);
        $resetTime = time() + (int) ceil(($this->limit - $remaining) / ($this->limit / 60));

        // Always send rate limit headers
        header('X-RateLimit-Limit: ' . $this->limit);
        header('X-RateLimit-Remaining: ' . $remaining);
        header('X-RateLimit-Reset: ' . $resetTime);

        if (!$allowed) {
            $retryAfter = max(1, $resetTime - time());
            http_response_code(429);
            header('Retry-After: ' . $retryAfter);

            return [
                'error'       => 'Too Many Requests',
                'retry_after' => $retryAfter,
            ];
        }

        return $next($request);
    }

    private function getClientId(Request $request): string {
        $user = $request->getAuthUser();
        return $user !== null ? 'user:' . $user->sub : 'ip:' . $request->ip();
    }
}
```

This middleware sends `X-RateLimit-*` headers on every response, enabling clients to monitor their usage and back off before hitting the limit.

## Common Pitfalls
1. **Only sending rate limit headers on 429 responses** — Clients need to see their remaining quota on every response so they can throttle proactively. Send `X-RateLimit-Limit`, `X-RateLimit-Remaining`, and `X-RateLimit-Reset` on all responses, not just 429s.
2. **Using seconds since epoch for Retry-After** — The `Retry-After` header can be either a number of seconds or an HTTP-date. Using seconds is simpler and avoids timezone confusion. Always prefer the integer format.

## Best Practices
1. **Include the limit in the error body too** — Not all HTTP clients make it easy to read response headers. Include `retry_after` and `limit` in the JSON body of the 429 response as well as in the headers.
2. **Use `#[\NoDiscard]` on rate limit check methods** — PHP 8.5’s `#[\NoDiscard]` attribute catches the common bug of calling `isAllowed()` or `consume()` without checking the return value. Apply it to any method whose return value must be used.

## Summary
- Send `X-RateLimit-Limit`, `X-RateLimit-Remaining`, and `X-RateLimit-Reset` headers on every API response.
- Send the `Retry-After` header only on 429 responses, preferring the integer (seconds) format.
- PHP 8.5’s `#[\NoDiscard]` attribute prevents the common bug of ignoring rate limit check results.
- Include rate limit information in both headers and the JSON response body for maximum client compatibility.
- Good rate limit communication reduces support burden and helps clients build robust retry logic.

## Code Examples

**Standard rate limit headers sent on every response, with Retry-After on 429s and PHP 8.5 #[\NoDiscard] to prevent unchecked rate limit results**

```php
<?php
// Send rate limit headers on EVERY response
header('X-RateLimit-Limit: 100');
header('X-RateLimit-Remaining: ' . $remaining);
header('X-RateLimit-Reset: ' . $resetTimestamp);

// On 429 responses, also send Retry-After
if ($remaining <= 0) {
    http_response_code(429);
    header('Retry-After: ' . max(1, $resetTimestamp - time()));
    echo json_encode([
        'error'       => 'Too Many Requests',
        'retry_after' => max(1, $resetTimestamp - time()),
    ]);
    exit;
}

// PHP 8.5: prevent ignored rate limit checks
class RateLimiter {
    #[\NoDiscard('Must check rate limit result')]
    public function isAllowed(string $clientId): bool {
        return $this->bucket->remaining($clientId) > 0;
    }
}
```


## Resources

- [PHP header function](https://www.php.net/manual/en/function.header.php) — Setting HTTP response headers for rate limit communication
- [PHP http_response_code function](https://www.php.net/manual/en/function.http-response-code.php) — Setting the HTTP status code including 429 Too Many Requests

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*