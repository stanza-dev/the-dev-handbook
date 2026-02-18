---
source_course: "go-architecture"
source_lesson: "go-architecture-functional-options-validation"
---

# Options that Return Errors

Sometimes options need validation. Use a modified pattern:

```go
type Option func(*Server) error

func WithPort(port int) Option {
    return func(s *Server) error {
        if port < 1 || port > 65535 {
            return fmt.Errorf("invalid port: %d", port)
        }
        s.Port = port
        return nil
    }
}

func NewServer(opts ...Option) (*Server, error) {
    s := &Server{Port: 8080}  // Default
    for _, opt := range opts {
        if err := opt(s); err != nil {
            return nil, err
        }
    }
    return s, nil
}
```

## Conditional Options

```go
func WithDebug(enable bool) Option {
    return func(s *Server) {
        if enable {
            s.Logger = debugLogger
            s.Verbose = true
        }
    }
}
```

## Code Examples

**TLS Option**

```go
// Options can depend on each other
func WithTLS(certFile, keyFile string) Option {
    return func(s *Server) error {
        cert, err := tls.LoadX509KeyPair(certFile, keyFile)
        if err != nil {
            return err
        }
        s.TLSConfig = &tls.Config{Certificates: []tls.Certificate{cert}}
        return nil
    }
}
```


---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*