---
source_course: "go-architecture"
source_lesson: "go-architecture-accept-interfaces"
---

# Accept Interfaces, Return Structs

## Introduction
"Accept interfaces, return structs" is one of Go's most important design principles. It maximizes flexibility for callers while keeping your API concrete and easy to understand. Following this pattern will make your Go code more testable and composable.

## Key Concepts
- **Accept Interfaces**: Function parameters should be interface types so callers can pass any implementation.
- **Return Structs**: Functions should return concrete types so callers know exactly what they get.
- **Implicit Satisfaction**: Go interfaces are satisfied implicitly—no `implements` keyword needed.

## Real World Context
Imagine you are building a service that reads user data. If your function accepts `*sql.DB`, it can only work with a real database. If it accepts an interface with a `QueryRow` method, you can pass a real database in production and a mock in tests.

## Deep Dive
When you accept an interface, you decouple from concrete implementations:

```go
// Good: Accepts any Reader
func Process(r io.Reader) error {
    // Works with files, network, buffers, etc.
    return nil
}

// Bad: Accepts concrete type
func Process(f *os.File) error {
    // Only works with files
    return nil
}
```

When you return a struct, callers get full access to the concrete type:

```go
// Good: Returns concrete type
func NewServer(addr string) *Server {
    return &Server{Addr: addr}
}

// Usually avoid: Returns interface
func NewServer(addr string) ServerInterface {
    return &server{addr: addr}
}
```

Returning structs is preferred because concrete types are easier to understand, callers do not need to look up what an interface contains, and the type is easier to extend without breaking changes. If a caller wants to use it as an interface, they can assign it to one.

## Common Pitfalls
1. **Accepting concrete types in library code** — This forces callers into a single implementation. Always prefer interfaces for dependencies you want to swap.
2. **Returning interfaces prematurely** — Only return an interface if you have a strong reason (e.g., hiding private fields across package boundaries). Otherwise, return the struct.

## Best Practices
1. **Define small interfaces at the call site** — The consumer should define only the methods it needs, not import a large interface from the producer.
2. **Use constructors that return pointers to structs** — `func NewFoo(...) *Foo` is the idiomatic Go constructor pattern.

## Summary
- Accept interface parameters for maximum flexibility.
- Return concrete struct types for clarity and extensibility.
- This pattern enables easy testing, dependency injection, and loose coupling.
- Let callers decide which interface to use when they need one.

## Code Examples

**A constructor that accepts a Logger interface for flexibility and returns a concrete *Service so callers know exactly what they receive**

```go
type Logger interface {
    Log(msg string)
}

// Accept interface for testability.
// Return concrete type for clarity.
func NewService(logger Logger) *Service {
    return &Service{logger: logger}
}
```


## Resources

- [Go Wiki — CodeReviewComments](https://github.com/golang/go/wiki/CodeReviewComments#interfaces) — Official Go code review guidance on interface usage

---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*