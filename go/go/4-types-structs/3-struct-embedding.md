---
source_course: "go"
source_lesson: "go-struct-embedding"
---

# Embedding (Composition)

Go uses embedding instead of inheritance. You embed one struct inside another to reuse fields and methods.

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

## Accessing Embedded Fields

```go
d := Dog{
    Animal: Animal{Name: "Buddy"},
    Breed:  "Golden Retriever",
}

// Promoted fields and methods
fmt.Println(d.Name)  // Same as d.Animal.Name
d.Speak()            // Calls Animal.Speak()
```

## Overriding Methods

You can define a method on the outer type to "override" the embedded method:

```go
func (d Dog) Speak() {
    fmt.Println(d.Name, "barks!")
}
```

## Multiple Embedding

```go
type ReadWriter struct {
    *Reader
    *Writer
}
```

If both embedded types have the same method, you must call it explicitly to avoid ambiguity.

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