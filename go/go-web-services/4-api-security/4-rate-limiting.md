---
source_course: "go-web-services"
source_lesson: "go-web-services-rate-limiting"
---

# Rate Limiting

## Introduction
Rate limiting protects your API from abuse, prevents resource exhaustion, and ensures fair access for all clients. Go's extended library provides a token bucket rate limiter out of the box.

## Key Concepts
- **Token bucket algorithm**: A rate limiting strategy where tokens are added at a fixed rate. Each request consumes a token; when the bucket is empty, requests are rejected.
- **rate.NewLimiter(rate, burst)**: Creates a limiter allowing `rate` requests per second with a `burst` size for temporary spikes.
- **Per-IP limiting**: Maintaining a separate rate limiter for each client IP address to prevent one client from consuming the entire quota.
- **HTTP 429 Too Many Requests**: The standard status code returned when a rate limit is exceeded.

## Real World Context
Without rate limiting, a single misbehaving client (or attacker) can overwhelm your API, causing downtime for everyone. Rate limiting is a required layer for any public-facing API and a best practice for internal services.

## Deep Dive

A global rate limiter using Go's `x/time/rate` package.

```go
import "golang.org/x/time/rate"

var limiter = rate.NewLimiter(rate.Limit(10), 30)  // 10 req/sec, burst 30

func rateLimitMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        if !limiter.Allow() {
            http.Error(w, "Too Many Requests", http.StatusTooManyRequests)
            return
        }
        next.ServeHTTP(w, r)
    })
}
```

`limiter.Allow()` returns false when the token bucket is empty, triggering a 429 response.

For per-IP rate limiting, maintain a map of limiters keyed by client IP.

```go
type ipLimiter struct {
    limiters map[string]*rate.Limiter
    mu       sync.Mutex
}

func (i *ipLimiter) getLimiter(ip string) *rate.Limiter {
    i.mu.Lock()
    defer i.mu.Unlock()

    if limiter, ok := i.limiters[ip]; ok {
        return limiter
    }

    limiter := rate.NewLimiter(rate.Limit(5), 10)
    i.limiters[ip] = limiter
    return limiter
}
```

This gives each IP its own bucket, so one abusive client does not affect others.

## Common Pitfalls
1. **Using a global limiter for all clients** — One client can consume the entire rate limit, blocking legitimate users. Use per-IP or per-API-key limiters.
2. **Not cleaning up stale limiters** — The per-IP map grows unbounded. Add a background goroutine to evict entries that have not been used recently.

## Best Practices
1. **Include a `Retry-After` header** — Tell rate-limited clients when they can retry, improving their experience and reducing retry storms.
2. **Use different limits for different endpoints** — Auth endpoints need stricter limits than read-only endpoints.

## Summary
- Use `golang.org/x/time/rate` for token bucket rate limiting in Go.
- Per-IP limiters prevent one client from monopolizing the API.
- Return HTTP 429 with a `Retry-After` header when rejecting rate-limited requests.

## Code Examples

**A per-IP rate limiting middleware with Retry-After header — each client IP gets its own token bucket to prevent one client from affecting others**

```go
func (rl *RateLimiter) Middleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        ip := getClientIP(r)
        limiter := rl.getLimiter(ip)

        if !limiter.Allow() {
            w.Header().Set("Retry-After", "60")
            http.Error(w, "Too Many Requests", http.StatusTooManyRequests)
            return
        }

        next.ServeHTTP(w, r)
    })
}
```


## Resources

- [x/time/rate Package](https://pkg.go.dev/golang.org/x/time/rate) — Official Go documentation for the token bucket rate limiter

---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*