---
source_course: "go-architecture"
source_lesson: "go-architecture-builder-pattern"
---

# Builder Pattern

An alternative to functional options for complex configurations.

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

## Usage

```go
server, err := NewServerBuilder().
    Port(8080).
    Timeout(10 * time.Second).
    Build()
```

## Code Examples

**Fluent Builder**

```go
// Fluent builder usage
client, err := NewHTTPClientBuilder().
    BaseURL("https://api.example.com").
    Timeout(30 * time.Second).
    RetryCount(3).
    WithAuth(token).
    Build()
```


---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*