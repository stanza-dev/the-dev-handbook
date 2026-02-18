---
source_course: "go"
source_lesson: "go-constructors-factories"
---

# Constructors in Go

Go does not have a special `constructor` keyword. Instead, the idiomatic way to initialize complex structs is via Factory functions, typically named `New` or `New[Type]`.

## The New Pattern

```go
type Server struct {
    Port int
    DB   *sql.DB
}

// NewServer creates and initializes a Server
func NewServer(port int) (*Server, error) {
    if port <= 0 {
        return nil, errors.New("invalid port")
    }
    return &Server{
        Port: port,
        DB:   nil, // explicitly or implicitly nil
    }, nil
}
```

## Why Return Pointers?

*   Avoids copying large structs.
*   Allows nil return for errors.
*   Matches the convention of methods using pointer receivers.

## Default Values

Use the factory to set sensible defaults:

```go
func NewConfig() *Config {
    return &Config{
        Timeout:    30 * time.Second,
        MaxRetries: 3,
        Debug:      false,
    }
}
```

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

- [Go Wiki - Constructor](https://go.dev/doc/effective_go#new) — Allocation with new and make

---

> 📘 *This lesson is part of the [Go Programming](https://stanza.dev/courses/go) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*