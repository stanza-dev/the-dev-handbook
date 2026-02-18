---
source_course: "go-architecture"
source_lesson: "go-architecture-accept-interfaces"
---

# The Pattern

"Accept interfaces, return structs" is a fundamental Go design principle.

## Accept Interfaces

```go
// Good: Accepts any Reader
func Process(r io.Reader) error {
    // Can work with files, network, buffers, etc.
}

// Bad: Accepts concrete type
func Process(f *os.File) error {
    // Only works with files
}
```

## Return Structs

```go
// Good: Returns concrete type
func NewServer(addr string) *Server {
    return &Server{Addr: addr}
}

// Usually Avoid: Returns interface
func NewServer(addr string) ServerInterface {
    return &server{addr: addr}
}
```

## Why Return Structs?

*   Concrete types are easier to understand.
*   No need to look up what the interface contains.
*   Callers can choose to use it as an interface if needed.
*   Easier to extend without breaking changes.

## Code Examples

**Pattern Example**

```go
type Logger interface {
    Log(msg string)
}

// Accept interface
func NewService(logger Logger) *Service {
    // Return concrete type
    return &Service{logger: logger}
}
```


---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*