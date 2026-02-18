---
source_course: "go-architecture"
source_lesson: "go-architecture-functional-options-basics"
---

# The Problem
Constructors with too many parameters (`NewServer(addr, port, timeout, logger, db, ...)`) are hard to read and maintain.

# The Solution
Use variadic functions that accept configuration closures.

```go
type Server struct {
    Timeout time.Duration
    Logger  Logger
}

type Option func(*Server)

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

// Usage
svc := NewServer(
    WithTimeout(5 * time.Second),
    WithLogger(myLogger),
)
```

## Code Examples

**Option function**

```go
func WithLogger(l *Logger) Option {
    return func(s *Server) {
        s.logger = l
    }
}
```


---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*