---
source_course: "go-architecture"
source_lesson: "go-architecture-interface-compliance"
---

# Compile-Time Interface Checks

Go's implicit interfaces mean you might not realize a type doesn't implement an interface until runtime. Use compile-time checks:

```go
// Ensure *MyServer implements http.Handler
var _ http.Handler = (*MyServer)(nil)

// Or more explicitly
var _ http.Handler = &MyServer{}
```

## When to Use

*   Exported types that should implement standard interfaces.
*   After refactoring to ensure you didn't break an implementation.
*   In libraries where interface compliance is part of the contract.

## Interface Assertions at Runtime

```go
func process(r io.Reader) {
    // Check if r also implements io.Closer
    if c, ok := r.(io.Closer); ok {
        defer c.Close()
    }
    // Process...
}
```

## Common Patterns

```go
// Check for optional interface
type Validator interface {
    Validate() error
}

func process(v any) error {
    if val, ok := v.(Validator); ok {
        if err := val.Validate(); err != nil {
            return err
        }
    }
    // Process...
}
```

## Code Examples

**Compile-Time Check**

```go
type Server struct{}

// Compile-time check
var _ http.Handler = (*Server)(nil)

func (s *Server) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    // ...
}
```


---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*