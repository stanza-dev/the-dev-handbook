---
source_course: "go-web-services"
source_lesson: "go-web-services-slog-context"
---

# Context-Aware Logging

## Introduction
In a web service handling many concurrent requests, you need every log line to carry a request ID and other metadata. Context-aware logging attaches these attributes once and propagates them through the entire request lifecycle.

## Key Concepts
- **slog.With()**: Creates a child logger with pre-set attributes that appear in every subsequent log call.
- **context.WithValue()**: Stores the logger in the request context so any function in the call chain can retrieve it.
- **Request ID**: A unique identifier (usually a UUID) attached to every log line for a given request, enabling end-to-end tracing.
- **Logger middleware**: A middleware that creates a per-request logger and injects it into the context.

## Real World Context
When debugging a production issue, you need to trace all log lines for a single request across multiple services. Context-aware logging with request IDs makes this possible by correlating logs from different parts of the system.

## Deep Dive

Create a logger with request-specific attributes using `slog.With()`.

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

    if err := process(); err != nil {
        logger.Error("processing failed", "error", err)
    }
}
```

Every log call on `logger` automatically includes the request ID, method, and path.

To make the logger available anywhere in the call chain, store it in the context.

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

The `FromContext` function falls back to the default logger if none is set, making it safe to call anywhere.

## Common Pitfalls
1. **Creating a new logger for every function** — This loses the shared attributes. Always retrieve the logger from the context instead.
2. **Using string keys for context values** — String keys can collide. Use unexported struct types like `ctxKey struct{}` to guarantee uniqueness.

## Best Practices
1. **Inject the logger via middleware** — Create the per-request logger in a single middleware and add it to the context for all downstream handlers.
2. **Always generate a request ID if one is missing** — If `X-Request-ID` is not in the incoming headers, generate a UUID to ensure every request is traceable.

## Summary
- Use `slog.With()` to create loggers with pre-set attributes like request ID.
- Store the logger in `context.Context` so the entire call chain can access it.
- A logging middleware creates the per-request logger and injects it into the context.

## Code Examples

**A logging middleware that creates a per-request logger with a unique ID and injects it into the context for downstream handlers**

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


## Resources

- [log/slog Package](https://pkg.go.dev/log/slog) — Official Go documentation covering slog.With and handler customization

---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*