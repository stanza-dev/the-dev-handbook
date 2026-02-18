---
source_course: "go-architecture"
source_lesson: "go-architecture-composition-over-inheritance"
---

# Composition

Go does not support type inheritance (subclassing). Instead, it uses composition via **embedding**.

```go
type Reader interface { Read(p []byte) (n int, err error) }
type Writer interface { Write(p []byte) (n int, err error) }

// Embedding interfaces
type ReadWriter interface {
    Reader
    Writer
}

// Embedding structs
type Base struct { ID int }
type User struct {
    Base // Promotes methods of Base to User
    Name string
}
```

## Interface Segregation
Prefer small interfaces. `io.Reader` has just one method. This makes it easy for many types to implement it, increasing reuse.

## Go Proverb
"The bigger the interface, the weaker the abstraction." - Rob Pike

## Code Examples

**Accept Interfaces**

```go
func Process(r io.Reader) {
    // I only need to Read, I don't care if it's a File, Network, or Buffer.
}
```


---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*