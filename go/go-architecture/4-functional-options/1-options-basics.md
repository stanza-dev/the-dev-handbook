---
source_course: "go-architecture"
source_lesson: "go-architecture-functional-options-basics"
---

# Functional Options

## Introduction
Constructors with many parameters are hard to read, maintain, and extend. The functional options pattern solves this by using variadic closure arguments that each configure one aspect of the object. This is one of Go's most celebrated API design patterns, used in production libraries like gRPC, Zap, and the AWS SDK.

## Key Concepts
- **Option Type**: A function type (e.g., `type Option func(*Server)`) that modifies the object being constructed.
- **With* Functions**: Named constructors like `WithTimeout()` that return an Option, making each configuration self-documenting.
- **Variadic Constructor**: A `New*` function that accepts `...Option`, allowing zero or more configuration options.

## Real World Context
You are building an HTTP client library. Users need to configure timeout, retry count, base URL, auth headers, and TLS settings. With a traditional constructor, `NewClient(url, timeout, retries, headers, tls)` becomes unreadable. Functional options let users write `NewClient(WithBaseURL(url), WithTimeout(5*time.Second))` — clear, flexible, and extensible.

## Deep Dive
The pattern has three parts. First, define the Option type:

```go
type Server struct {
    Timeout time.Duration
    Logger  Logger
}

type Option func(*Server)
```

Second, create With* functions for each configurable field:

```go
func WithTimeout(d time.Duration) Option {
    return func(s *Server) {
        s.Timeout = d
    }
}

func WithLogger(l Logger) Option {
    return func(s *Server) {
        s.Logger = l
    }
}
```

Third, the constructor applies defaults then options:

```go
func NewServer(opts ...Option) *Server {
    s := &Server{
        Timeout: 30 * time.Second,  // Default
        Logger:  defaultLogger,
    }
    for _, opt := range opts {
        opt(s)
    }
    return s
}
```

Callers compose exactly what they need:

```go
svc := NewServer(
    WithTimeout(5 * time.Second),
    WithLogger(myLogger),
)
```

## Common Pitfalls
1. **Not setting defaults before applying options** — Always initialize the struct with sensible defaults first. Options override them.
2. **Creating options that conflict** — Document when two options are mutually exclusive (e.g., WithTLS and WithInsecure).

## Best Practices
1. **Name options with the With prefix** — `WithTimeout`, `WithLogger`, `WithRetry`. This is the universal Go convention.
2. **Keep the default constructor usable** — `NewServer()` with zero options should produce a working server with sensible defaults.

## Summary
- Functional options replace long parameter lists with named, composable configuration.
- Define `type Option func(*T)` and `With*` functions for each setting.
- Always set defaults before applying options.
- The pattern enables backwards-compatible API evolution—new options never break existing callers.

## Code Examples

**A With* option function that configures the server's logger — callers compose these options to customize construction**

```go
// WithLogger returns an Option that sets the server's logger.
// Each With* function configures one aspect of the server.
func WithLogger(l *Logger) Option {
    return func(s *Server) {
        s.logger = l
    }
}

// Usage: NewServer(WithLogger(myLogger), WithTimeout(5*time.Second))
```


## Resources

- [Go Blog — Self-referential functions and the design of options](https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design-of.html) — Rob Pike's original blog post introducing the functional options pattern
- [Effective Go](https://go.dev/doc/effective_go) — Official Go guide covering idiomatic patterns and API design

---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*