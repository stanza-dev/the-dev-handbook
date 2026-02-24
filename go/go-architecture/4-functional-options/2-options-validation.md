---
source_course: "go-architecture"
source_lesson: "go-architecture-functional-options-validation"
---

# Options with Validation

## Introduction
Not all configuration values are valid. A port number must be 1-65535, a timeout must be positive, and a TLS certificate must load correctly. Extending the functional options pattern to return errors lets you validate each option at construction time, catching misconfigurations early.

## Key Concepts
- **Error-Returning Option**: Changing the Option type to `func(*Server) error` so each option can report validation failures.
- **Conditional Option**: An option that only applies configuration when a condition is true.
- **Early Validation**: Checking configuration at construction time rather than at first use.

## Real World Context
A user passes `WithPort(-1)` to your constructor. Without validation, this creates a server that fails when you call `ListenAndServe`. With error-returning options, the constructor returns an error immediately with a clear message: "invalid port: -1".

## Deep Dive
Modify the Option type to return an error:

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
```

The constructor now checks each option:

```go
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

Conditional options allow feature toggles:

```go
func WithDebug(enable bool) Option {
    return func(s *Server) error {
        if enable {
            s.Logger = debugLogger
            s.Verbose = true
        }
        return nil
    }
}
```

Options can perform complex operations like loading TLS certificates:

```go
func WithTLS(certFile, keyFile string) Option {
    return func(s *Server) error {
        cert, err := tls.LoadX509KeyPair(certFile, keyFile)
        if err != nil {
            return fmt.Errorf("loading TLS cert: %w", err)
        }
        s.TLSConfig = &tls.Config{
            Certificates: []tls.Certificate{cert},
        }
        return nil
    }
}
```

## Common Pitfalls
1. **Ignoring option errors** — When using error-returning options, always check the error from the constructor. A silent misconfiguration is worse than a crash.
2. **Validating too late** — Validate in the option function, not when the setting is first used. Fail fast at construction.

## Best Practices
1. **Wrap errors with context** — Use `fmt.Errorf("loading TLS cert: %w", err)` so callers know which option failed.
2. **Keep validation in the option** — Each With* function is responsible for validating its own input.

## Summary
- Error-returning options catch invalid configuration at construction time.
- The constructor stops at the first error, providing clear feedback.
- Conditional options enable feature toggles and environment-specific configuration.
- Complex options can perform I/O like loading certificates.

## Code Examples

**A TLS option that validates and loads certificates at construction time, returning a wrapped error if loading fails**

```go
// WithTLS loads a TLS certificate and configures the server.
// It returns an error if the certificate cannot be loaded.
func WithTLS(certFile, keyFile string) Option {
    return func(s *Server) error {
        cert, err := tls.LoadX509KeyPair(certFile, keyFile)
        if err != nil {
            return fmt.Errorf("loading TLS cert: %w", err)
        }
        s.TLSConfig = &tls.Config{
            Certificates: []tls.Certificate{cert},
        }
        return nil
    }
}
```


## Resources

- [Functional Options for Friendly APIs](https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis) — Dave Cheney's influential post on functional options with validation

---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*