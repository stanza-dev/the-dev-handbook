---
source_course: "go-architecture"
source_lesson: "go-architecture-builder-pattern"
---

# Builder Pattern Alternative

## Introduction
The builder pattern is an alternative to functional options for complex configurations. It uses method chaining (fluent API) and defers validation to a final `Build()` call. Understanding both patterns lets you choose the right one for each situation.

## Key Concepts
- **Builder Struct**: An intermediate struct that accumulates configuration before building the final object.
- **Method Chaining**: Each setter method returns the builder itself, enabling `b.Port(8080).Timeout(5s).Build()`.
- **Deferred Validation**: All validation happens in `Build()`, which can check interdependent fields together.

## Real World Context
You are configuring an HTTP client with a base URL, authentication, retry policy, and circuit breaker. Some settings depend on others—retries require a timeout, and the circuit breaker threshold depends on the retry count. A builder can validate these relationships in `Build()`.

## Deep Dive
The builder accumulates state and validates at build time:

```go
type ServerBuilder struct {
    port    int
    timeout time.Duration
    logger  Logger
    err     error
}

func NewServerBuilder() *ServerBuilder {
    return &ServerBuilder{
        port:    8080,
        timeout: 30 * time.Second,
    }
}

func (b *ServerBuilder) Port(p int) *ServerBuilder {
    if p < 1 || p > 65535 {
        b.err = fmt.Errorf("invalid port: %d", p)
    }
    b.port = p
    return b
}

func (b *ServerBuilder) Timeout(d time.Duration) *ServerBuilder {
    b.timeout = d
    return b
}

func (b *ServerBuilder) Build() (*Server, error) {
    if b.err != nil {
        return nil, b.err
    }
    return &Server{Port: b.port, Timeout: b.timeout}, nil
}
```

Usage is fluent:

```go
server, err := NewServerBuilder().
    Port(8080).
    Timeout(10 * time.Second).
    Build()
```

Compared to functional options, the builder is better when you need complex interdependent validation (Build() can check all fields together) and a fluent API style. Functional options are better for backwards-compatible API evolution and simpler cases.

## Common Pitfalls
1. **Continuing after an error** — The builder stores the first error and returns it in Build(). Subsequent setters may still run but their values are irrelevant.
2. **Forgetting to call Build()** — The builder is not the final object. Always call Build() to get the validated result.

## Best Practices
1. **Use builder for complex interdependent validation** — When settings depend on each other, Build() can validate the complete configuration.
2. **Store the first error** — Use a single `err` field on the builder. Stop meaningful work after the first error.

## Summary
- The builder pattern uses method chaining and deferred validation.
- Build() validates the complete configuration and returns the final object or an error.
- Prefer builder when configuration fields are interdependent.
- Prefer functional options when you need backwards-compatible API evolution.

## Code Examples

**A fluent builder API for an HTTP client showing method chaining — Build() is called last to validate and construct the final object**

```go
// Fluent builder usage for an HTTP client.
// Each method returns the builder for chaining.
// Build() validates and returns the final client.
client, err := NewHTTPClientBuilder().
    BaseURL("https://api.example.com").
    Timeout(30 * time.Second).
    RetryCount(3).
    WithAuth(token).
    Build()
```


## Resources

- [Effective Go](https://go.dev/doc/effective_go) — Official Go guide covering constructor and initialization patterns

---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*