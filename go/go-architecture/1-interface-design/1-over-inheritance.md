---
source_course: "go-architecture"
source_lesson: "go-architecture-composition-over-inheritance"
---

# Composition over Inheritance

## Introduction
Go deliberately omits class-based inheritance. Instead of building deep type hierarchies, Go developers compose small, focused types together using embedding. Understanding this shift is essential because it shapes every design decision you make in Go.

## Key Concepts
- **Embedding**: Including one type inside another so its fields and methods are promoted to the outer type.
- **Interface Segregation**: Keeping interfaces as small as possible so many types can satisfy them.
- **Method Promotion**: When you embed a struct or interface, its methods become callable directly on the embedding type.

## Real World Context
In languages like Java, you might create a `BaseEntity` class and extend it everywhere. In Go, you embed a `Base` struct and get the same code reuse without coupling your entire hierarchy. The standard library uses this pattern extensively—`io.ReadWriter` embeds `io.Reader` and `io.Writer`.

## Deep Dive
Go provides two forms of embedding: interface embedding and struct embedding.

Interface embedding lets you combine small interfaces into larger ones:

```go
type Reader interface { Read(p []byte) (n int, err error) }
type Writer interface { Write(p []byte) (n int, err error) }

// ReadWriter is the composition of Reader and Writer
type ReadWriter interface {
    Reader
    Writer
}
```

Any type that has both `Read` and `Write` methods automatically satisfies `ReadWriter`. There is no explicit declaration needed.

Struct embedding promotes fields and methods from the inner type:

```go
type Base struct { ID int }
type User struct {
    Base // Promotes ID field and any Base methods
    Name string
}

u := User{Base: Base{ID: 1}, Name: "Alice"}
fmt.Println(u.ID) // Accessing promoted field directly
```

The Go proverb states: "The bigger the interface, the weaker the abstraction." `io.Reader` has one method, which means files, network connections, buffers, and compressors all implement it.

## Common Pitfalls
1. **Creating large interfaces upfront** — Resist the urge to define a 10-method interface. Start with one or two methods and grow only when needed.
2. **Confusing embedding with inheritance** — Embedding is not subtyping. An embedded type's methods are promoted, but the outer type is not a subtype of the inner type.

## Best Practices
1. **Start with small interfaces** — One or two methods is ideal. The standard library averages 1.3 methods per interface.
2. **Compose interfaces from smaller ones** — Build larger contracts by embedding small interfaces, just like `io.ReadWriteCloser`.

## Summary
- Go uses composition via embedding instead of class inheritance.
- Interface embedding creates larger contracts from small ones.
- Struct embedding promotes fields and methods to the outer type.
- Smaller interfaces lead to stronger abstractions and more reusable code.

## Code Examples

**Accepting the io.Reader interface demonstrates how small interfaces enable maximum flexibility — any type with a Read method works here**

```go
// Accept the smallest interface you need.
// This function works with files, network connections, buffers, etc.
func Process(r io.Reader) {
    buf := make([]byte, 1024)
    n, err := r.Read(buf)
    fmt.Printf("Read %d bytes\n", n)
}
```


## Resources

- [Effective Go — Embedding](https://go.dev/doc/effective_go#embedding) — Official guide to embedding structs and interfaces in Go
- [Go Proverbs](https://go-proverbs.github.io/) — Rob Pike's Go proverbs including the interface abstraction principle

---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*