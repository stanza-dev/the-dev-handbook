---
source_course: "go-architecture"
source_lesson: "go-architecture-config-structs"
---

# Simple Config Struct

For simpler cases, a config struct might be cleaner than functional options:

```go
type ServerConfig struct {
    Port    int
    Timeout time.Duration
    Logger  Logger
    // Use pointer for "optional" fields
    TLS     *TLSConfig
}

func NewServer(cfg ServerConfig) (*Server, error) {
    // Apply defaults for zero values
    if cfg.Port == 0 {
        cfg.Port = 8080
    }
    if cfg.Timeout == 0 {
        cfg.Timeout = 30 * time.Second
    }
    return &Server{...}, nil
}
```

## Comparison

| Pattern | Best For |
|---------|----------|
| Config Struct | Few options, simple types |
| Functional Options | Many options, complex defaults |
| Builder | Complex validation, fluent API |

## Combining Patterns

```go
func NewServer(cfg ServerConfig, opts ...Option) *Server {
    s := &Server{...from cfg...}
    for _, opt := range opts {
        opt(s)
    }
    return s
}
```

## Code Examples

**Zero Value Config**

```go
// Config struct with sensible defaults
type Config struct {
    Port       int           // 0 means use default (8080)
    Timeout    time.Duration // 0 means use default (30s)
    MaxRetries int           // 0 means no retries
    Debug      bool          // false is the default
}

// Zero value is valid configuration
server := NewServer(Config{})  // All defaults
```


---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*