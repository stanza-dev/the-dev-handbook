---
source_course: "go-web-services"
source_lesson: "go-web-services-http-middleware"
---

# Middleware Patterns

## Introduction
Middleware is the backbone of request processing pipelines in Go. It lets you execute cross-cutting logic like logging, authentication, and CORS before or after a handler runs.

## Key Concepts
- **Middleware function**: A function that takes an `http.Handler` and returns a new `http.Handler`, wrapping the original with additional behavior.
- **Handler chain**: Multiple middlewares composed together, each wrapping the next, forming a pipeline.
- **next.ServeHTTP(w, r)**: The call that delegates processing to the next handler in the chain.
- **ResponseWriter wrapper**: A custom type embedding `http.ResponseWriter` to capture response metadata like status codes.

## Real World Context
Every production Go service uses middleware for logging, authentication, tracing, and rate limiting. Understanding the wrapper pattern is essential because it is the standard way to add cross-cutting concerns without modifying individual handlers.

## Deep Dive

A middleware is a function that accepts a handler and returns a new handler. Code before `next.ServeHTTP` runs on the request path; code after runs on the response path.

```go
func LoggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        next.ServeHTTP(w, r)
        fmt.Printf("%s %s %v\n", r.Method, r.URL.Path, time.Since(start))
    })
}
```

The `fmt.Printf` line only executes after the inner handler has finished writing the response.

Chain multiple middlewares by nesting them. The outermost middleware runs first.

```go
handler := LoggingMiddleware(AuthMiddleware(finalHandler))
```

This means logging wraps auth, which wraps the final handler. Requests flow inward; responses flow outward.

To capture the HTTP status code (which `ResponseWriter` does not expose after writing), wrap it in a custom type.

```go
type responseWriter struct {
    http.ResponseWriter
    status int
}

func (rw *responseWriter) WriteHeader(code int) {
    rw.status = code
    rw.ResponseWriter.WriteHeader(code)
}
```

This lets logging middleware report the actual status code returned by the handler.

## Common Pitfalls
1. **Calling `next.ServeHTTP` after writing a response** — If you write an error (e.g., 401) and then still call `next.ServeHTTP`, the handler runs anyway, causing duplicate writes.
2. **Not deferring post-processing** — If the handler panics, code after `next.ServeHTTP` never runs. Use `defer` for critical cleanup like logging.

## Best Practices
1. **Keep middlewares small and focused** — Each middleware should do one thing (logging, auth, CORS). Compose them rather than creating monolithic wrappers.
2. **Use `defer` for post-processing logic** — This ensures metrics and logging run even if a downstream handler panics.

## Summary
- Middleware wraps handlers using the `func(http.Handler) http.Handler` pattern.
- Code before `next.ServeHTTP` is pre-processing; code after is post-processing.
- Chain middlewares by nesting: `Logging(Auth(handler))`.

## Code Examples

**An authentication middleware that checks the Authorization header — returning early without calling next.ServeHTTP stops the chain**

```go
func AuthMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        token := r.Header.Get("Authorization")
        if token == "" {
            http.Error(w, "Unauthorized", http.StatusUnauthorized)
            return
        }
        next.ServeHTTP(w, r)
    })
}
```


## Resources

- [Effective Go](https://go.dev/doc/effective_go) — Official Go guide covering idiomatic patterns including interfaces and composition
- [net/http Package](https://pkg.go.dev/net/http) — Official Go documentation for HTTP handler and middleware types

---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*