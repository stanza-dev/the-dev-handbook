---
source_course: "go"
source_lesson: "go-constructors-factories"
---

# Constructors & Factories

## Introduction

Go does not have a special `constructor` keyword or a `new` class mechanism like Java or Python. Instead, the idiomatic way to initialize complex structs is through factory functions, typically named `New` or `New[Type]`. This pattern gives you full control over validation, defaults, and returned types.

## Key Concepts

- **Factory Function**: A regular function that creates and returns an initialized struct, conventionally named `NewTypeName`.
- **Returning Pointers**: Factories typically return `*T` to avoid large copies and allow nil error returns.
- **Default Values**: Factories set sensible defaults for fields the caller does not need to specify.
- **Functional Options**: An advanced pattern for configurable constructors using variadic option functions.

## Real World Context

Every major Go project uses this pattern. The standard library has `http.NewServeMux()`, `bufio.NewReader()`, and `errors.New()`. In application code, constructors validate configuration, establish database connections, and wire up dependencies. When a package exports only one main type, the convention is to name the factory simply `New()`.

## Deep Dive

The typical constructor returns a pointer and optionally an error.

```go
type Server struct {
    Port int
    DB   *sql.DB
}

func NewServer(port int) (*Server, error) {
    if port <= 0 {
        return nil, errors.New("invalid port")
    }
    return &Server{
        Port: port,
        DB:   nil,
    }, nil
}
```

Returning a pointer is idiomatic because it avoids copying large structs, allows nil to signal failure, and matches the convention of pointer receiver methods.

### Setting Default Values

Factories are the right place to assign sensible defaults.

```go
func NewConfig() *Config {
    return &Config{
        Timeout:    30 * time.Second,
        MaxRetries: 3,
        Debug:      false,
    }
}
```

Callers can override specific fields after construction when needed.

### Naming Conventions

The standard library establishes a clear naming pattern:

- `NewTypeName` when the package exports multiple types (`http.NewServeMux()`).
- `New` when the package exports one primary type (`errors.New()`, `ring.New()`).

Following this convention makes your API instantly familiar to other Go developers.

## Common Pitfalls

1. **Skipping validation in constructors** — Letting invalid state pass through the factory pushes errors deeper into the program where they are harder to debug.
2. **Returning values instead of pointers for large structs** — This causes unnecessary copying and prevents nil-based error signaling.

### Enhanced `new()` in Go 1.26\n\nGo 1.26 introduced an important quality-of-life improvement: the `new()` builtin now accepts expressions to initialize the value. Previously, creating a pointer to a simple value required a temporary variable:\n\n```go\n// Before Go 1.26 — needed a temp variable\nage := 25\nuser := User{Age: &age}\n```\n\nNow you can write this directly:\n\n```go\n// Go 1.26+ — new() accepts expressions\nuser := User{Age: new(25)}\nuser2 := User{Age: new(yearsSince(born))}\n```\n\nThis eliminates the common annoyance of needing temporary variables just to take the address of a value, especially useful for optional pointer fields in JSON structs and protobuf messages.\n\n## Best Practices

1. **Always validate inputs in the factory** — Return an error early if the configuration is invalid rather than panicking later.
2. **Follow the `NewTypeName` naming convention** — Consistency with the standard library makes your API predictable and discoverable.

## Summary

- Go uses factory functions (`NewTypeName`) instead of class constructors.
- Factories return pointers and optional errors for clean initialization.
- Use factories to set default values and validate inputs.
- Follow standard library naming: `NewX` for single-type packages, `NewTypeName` for multi-type packages.
- The functional options pattern extends constructors for complex configuration needs.

## Code Examples

**Constructor Pattern**

```go
package main

type File struct {
    Name string
    Mode os.FileMode
}

func NewFile(name string) *File {
    return &File{
        Name: name,
        Mode: 0644, // Default permissions
    }
}

func main() {
    f := NewFile("data.txt")
    fmt.Println(f.Name)
}
```


## Resources

- [Effective Go - Allocation with new](https://go.dev/doc/effective_go#new) — Official Effective Go guide section on allocation and the new function

---

> 📘 *This lesson is part of the [Go Programming](https://stanza.dev/courses/go) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*