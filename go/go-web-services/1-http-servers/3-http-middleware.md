---
source_course: "go-web-services"
source_lesson: "go-web-services-http-middleware"
---

# Middleware

Middleware wraps a handler to execute code before or after the request processing (e.g., logging, auth, CORS).

## The Pattern
A middleware is a function that takes a Handler and returns a Handler.

```go
func LoggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        next.ServeHTTP(w, r)
        fmt.Printf("%s %s %v\n", r.Method, r.URL.Path, time.Since(start))
    })
}
```

## Chaining
You can chain multiple middlewares:

```go
handler := LoggingMiddleware(AuthMiddleware(finalHandler))
```

## Response Capture

To log status codes, wrap ResponseWriter:

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

## Code Examples

**Auth Middleware**

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


---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*