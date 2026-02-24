---
source_course: "go"
source_lesson: "go-struct-embedding"
---

# Struct Embedding

## Introduction

Go does not have inheritance, but it offers something equally powerful: embedding. By embedding one struct inside another, you gain access to the embedded type's fields and methods as if they were your own. This composition-based approach is a cornerstone of idiomatic Go design.

## Key Concepts

- **Embedding**: Declaring a type as an anonymous field inside another struct, promoting its fields and methods.
- **Promotion**: Embedded fields and methods become directly accessible on the outer struct.
- **Composition over Inheritance**: Go's design philosophy favoring combining simple types rather than building deep class hierarchies.
- **Shadowing**: Defining a method on the outer type with the same name as an embedded method, effectively overriding it.

## Real World Context

Embedding is used extensively in production Go code. The standard library embeds `sync.Mutex` into structs that need locking, `http.Handler` implementations compose middleware layers through embedding, and ORM libraries embed base model structs into domain entities. Understanding embedding is essential for reading and writing idiomatic Go.

## Deep Dive

Embedding places one struct inside another without giving it a field name.

```go
type Animal struct {
    Name string
}

func (a Animal) Speak() {
    fmt.Println(a.Name, "makes a sound")
}

type Dog struct {
    Animal  // Embedded (anonymous field)
    Breed string
}
```

The `Dog` type now has direct access to `Name` and `Speak()` without qualification.

### Accessing Embedded Fields

Promoted fields and methods can be accessed directly on the outer struct.

```go
d := Dog{
    Animal: Animal{Name: "Buddy"},
    Breed:  "Golden Retriever",
}

fmt.Println(d.Name)  // Same as d.Animal.Name
d.Speak()            // Calls Animal.Speak()
```

Both the short form (`d.Name`) and the explicit form (`d.Animal.Name`) are valid. The explicit form is required when there is ambiguity.

### Overriding Embedded Methods

You can define a method on the outer type to shadow the embedded method.

```go
func (d Dog) Speak() {
    fmt.Println(d.Name, "barks!")
}
```

Now calling `d.Speak()` invokes the Dog method. You can still call `d.Animal.Speak()` for the original.

### Multiple Embedding

A struct can embed multiple types.

```go
type ReadWriter struct {
    *Reader
    *Writer
}
```

If both embedded types share a method name, you must call it explicitly to avoid ambiguity (`rw.Reader.Close()` vs `rw.Writer.Close()`).

## Common Pitfalls

1. **Ambiguous selectors with multiple embeds** — When two embedded types share a field or method name, Go raises a compile error unless you qualify the access explicitly.
2. **Assuming embedding is inheritance** — Embedded methods receive the embedded type as their receiver, not the outer type. The embedded `Animal.Speak()` method does not know about `Dog`.

## Best Practices

1. **Favor composition over deep nesting** — Embed only what you need and keep the hierarchy shallow for readability.
2. **Use embedding to implement interfaces** — Embedding a type that satisfies an interface makes the outer type satisfy it too, which is a clean pattern for decorators and middleware.

## Summary

- Embedding places a type as an anonymous field, promoting its fields and methods to the outer struct.
- Promoted members are accessed directly or via the explicit path.
- You can shadow embedded methods by defining the same method on the outer type.
- Multiple embedding is supported but requires explicit qualification for ambiguous names.
- Embedding is composition, not inheritance: the embedded receiver is always the inner type.

## Code Examples

**Struct Embedding**

```go
type Base struct {
    ID int
}

func (b Base) String() string {
    return fmt.Sprintf("ID: %d", b.ID)
}

type User struct {
    Base  // Embeds ID and String()
    Name string
}

func main() {
    u := User{Base: Base{ID: 1}, Name: "Alice"}
    fmt.Println(u.ID)       // 1
    fmt.Println(u.String()) // ID: 1
}
```


## Resources

- [Effective Go - Embedding](https://go.dev/doc/effective_go#embedding) — Official guide on struct embedding

---

> 📘 *This lesson is part of the [Go Programming](https://stanza.dev/courses/go) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*