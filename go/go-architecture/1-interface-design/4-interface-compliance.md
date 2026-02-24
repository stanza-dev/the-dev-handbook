---
source_course: "go-architecture"
source_lesson: "go-architecture-interface-compliance"
---

# Interface Compliance & Assertions

## Introduction
Go's implicit interface satisfaction is powerful but can be a double-edged sword: you might not realize a type has stopped implementing an interface until runtime. Compile-time checks and runtime type assertions give you safety nets. Every Go developer should know these techniques.

## Key Concepts
- **Compile-Time Interface Check**: A `var _ Interface = (*Type)(nil)` declaration that fails at compile time if the type does not satisfy the interface.
- **Type Assertion**: A runtime check (`v.(Type)`) that tests whether a value implements a specific interface or is a specific type.
- **Comma-Ok Idiom**: Using `v, ok := x.(Type)` to safely attempt a type assertion without panicking.

## Real World Context
You refactor a method signature on your `Server` type and accidentally break its `http.Handler` implementation. Without a compile-time check, you would not discover this until the code runs and panics. A single-line compile-time assertion catches this immediately.

## Deep Dive
Compile-time interface checks use a blank identifier assignment:

```go
// Ensure *MyServer implements http.Handler at compile time
var _ http.Handler = (*MyServer)(nil)
```

If `*MyServer` is missing the `ServeHTTP` method, the compiler produces a clear error. This is especially valuable for exported types in libraries.

Runtime type assertions let you check for optional interfaces:

```go
func process(r io.Reader) {
    // Check if r also implements io.Closer
    if c, ok := r.(io.Closer); ok {
        defer c.Close()
    }
    // Continue processing with r...
}
```

You can also check for custom optional behavior:

```go
type Validator interface {
    Validate() error
}

func process(v any) error {
    if val, ok := v.(Validator); ok {
        if err := val.Validate(); err != nil {
            return err
        }
    }
    return nil
}
```

This pattern is used throughout the standard library—for example, `io.Copy` checks if the reader implements `WriterTo` for an optimized path.

## Common Pitfalls
1. **Forgetting compile-time checks after refactoring** — Add `var _ Interface = (*Type)(nil)` for all exported types that must satisfy key interfaces.
2. **Using type assertions without the comma-ok form** — `v.(Type)` panics if the assertion fails. Always use `v, ok := x.(Type)` unless you are certain of the type.

## Best Practices
1. **Add compile-time checks for all public interface contracts** — Place them near the type definition so they are easy to find.
2. **Use type assertions for optional behavior** — Check for additional interfaces at runtime to provide optimized or extended behavior without requiring it.

## Summary
- Use `var _ Interface = (*Type)(nil)` to catch broken interface implementations at compile time.
- Use the comma-ok pattern `v, ok := x.(Type)` for safe runtime type assertions.
- Compile-time checks are essential after refactoring exported types.
- Runtime assertions enable optional interface patterns used throughout the standard library.

## Code Examples

**A compile-time interface check ensures *Server always implements http.Handler — if you accidentally remove or rename ServeHTTP, the build fails immediately**

```go
type Server struct{}

// Compile-time check: fails to build if Server
// does not implement http.Handler
var _ http.Handler = (*Server)(nil)

func (s *Server) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    // Handle the request
}
```


## Resources

- [Effective Go — Interface checks](https://go.dev/doc/effective_go#interface_conversions) — Official guidance on interface conversions and type assertions

---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*