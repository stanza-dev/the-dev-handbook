---
source_course: "go-architecture"
source_lesson: "go-architecture-config-structs"
---

# Config Struct Pattern

## Introduction
For simpler configurations, a plain config struct can be cleaner than functional options or builders. This pattern uses Go's zero values as defaults, making the most common configuration require zero effort. Understanding all three patterns lets you pick the right tool for each situation.

## Key Concepts
- **Zero Value Defaults**: Go initializes all fields to their zero values. Design your struct so zero values are sensible defaults.
- **Pointer Fields for Optional Settings**: Use pointer types to distinguish between "not set" and "set to zero".
- **Config + Options Hybrid**: Combining a config struct for required settings with functional options for optional ones.

## Real World Context
Your internal service has 3-4 settings: port, timeout, debug mode. Functional options would be over-engineering. A simple config struct with zero-value defaults lets callers write `NewServer(Config{})` for all defaults or `NewServer(Config{Port: 9090})` to override just one.

## Deep Dive
A config struct relies on zero values:

```go
type ServerConfig struct {
    Port    int
    Timeout time.Duration
    Logger  Logger
    TLS     *TLSConfig  // Pointer: nil means disabled
}

func NewServer(cfg ServerConfig) (*Server, error) {
    if cfg.Port == 0 {
        cfg.Port = 8080  // Default
    }
    if cfg.Timeout == 0 {
        cfg.Timeout = 30 * time.Second  // Default
    }
    return &Server{port: cfg.Port, timeout: cfg.Timeout}, nil
}
```

The zero value `Config{}` produces a working server with all defaults. Each pattern has its sweet spot:

| Pattern | Best For |
|---------|----------|
| Config Struct | Few options, simple types |
| Functional Options | Many options, complex defaults, library APIs |
| Builder | Complex validation, fluent API, interdependent fields |

You can combine patterns for maximum flexibility:

```go
func NewServer(cfg ServerConfig, opts ...Option) *Server {
    s := &Server{port: cfg.Port, timeout: cfg.Timeout}
    for _, opt := range opts {
        opt(s)
    }
    return s
}
```

## Common Pitfalls
1. **Ambiguous zero values** — If `Port: 0` could mean "use default" or "use port 0", use a pointer `*int` to distinguish nil from zero.
2. **Too many fields** — If your config struct has 15+ fields, functional options or a builder pattern might be clearer.

## Best Practices
1. **Design for zero-value usability** — `NewServer(Config{})` should work. Do not require callers to set fields for basic usage.
2. **Use pointer fields for truly optional settings** — `TLS *TLSConfig` clearly communicates that TLS is optional.

## Summary
- Config structs work best for simple configurations with few options.
- Design structs so zero values are sensible defaults.
- Use pointer fields to distinguish "not set" from "set to zero".
- Combine config structs with functional options when you need both required and optional settings.

## Code Examples

**A config struct where every zero value maps to a sensible default, so callers can start with an empty struct and override only what they need**

```go
// Config struct with sensible zero-value defaults.
// Callers can create a working server with just Config{}.
type Config struct {
    Port       int           // 0 means use default (8080)
    Timeout    time.Duration // 0 means use default (30s)
    MaxRetries int           // 0 means no retries
    Debug      bool          // false is the default
}

// Zero value is valid configuration
server := NewServer(Config{})  // All defaults applied
```


## Resources

- [Go Blog — The zero value](https://go.dev/ref/spec#The_zero_value) — Go specification section on zero values and their role in idiomatic Go design

---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*