---
source_course: "go-web-services"
source_lesson: "go-web-services-rate-limiting"
---

# Protecting APIs

Rate limiting prevents abuse and ensures fair usage.

## Token Bucket with golang.org/x/time/rate

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

## Per-IP Rate Limiting

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

## Code Examples

**IP-Based Rate Limiter**

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


---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*