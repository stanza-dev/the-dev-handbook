---
source_course: "go-web-services"
source_lesson: "go-web-services-slog-context"
---

# Logger with Context

Add common attributes to all logs in a request:

```go
func requestLogger(r *http.Request) *slog.Logger {
    return slog.With(
        "request_id", r.Header.Get("X-Request-ID"),
        "method", r.Method,
        "path", r.URL.Path,
    )
}

func handler(w http.ResponseWriter, r *http.Request) {
    logger := requestLogger(r)
    logger.Info("handling request")
    
    // Use logger throughout the handler
    if err := process(); err != nil {
        logger.Error("processing failed", "error", err)
    }
}
```

## Logger in Context

```go
type ctxKey struct{}

func WithLogger(ctx context.Context, logger *slog.Logger) context.Context {
    return context.WithValue(ctx, ctxKey{}, logger)
}

func FromContext(ctx context.Context) *slog.Logger {
    if logger, ok := ctx.Value(ctxKey{}).(*slog.Logger); ok {
        return logger
    }
    return slog.Default()
}
```

## Code Examples

**Logging Middleware**

```go
func loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        logger := slog.With(
            "request_id", uuid.New().String(),
            "method", r.Method,
            "path", r.URL.Path,
        )
        ctx := WithLogger(r.Context(), logger)
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}
```


---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*