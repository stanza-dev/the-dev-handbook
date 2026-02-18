---
source_course: "go"
source_lesson: "go-interfaces-implicit"
---

# Interfaces

In Go, interfaces are satisfied **implicitly**. If a type implements all the methods of an interface, it implements the interface. There is no `implements` keyword.

```go
type Stringer interface {
    String() string
}

type Person struct {
    Name string
}

// Person implements Stringer implicitly
func (p Person) String() string {
    return p.Name
}
```

## Interface Values

An interface value holds a concrete type and a value of that type:

```go
var s Stringer
s = Person{"Alice"}
fmt.Println(s.String())  // Alice
```

## Nil Interface Values

An interface value is nil only if both its type and value are nil:

```go
var s Stringer      // nil interface
var p *Person       // nil pointer
s = p              // s is NOT nil! (has type *Person)
```

This is a common source of bugs!

## Code Examples

**Interface Usage**

```go
type Shape interface {
    Area() float64
}

type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}

func printArea(s Shape) {
    fmt.Printf("Area: %f\n", s.Area())
}
```


## Resources

- [A Tour of Go - Interfaces](https://go.dev/tour/methods/9) — Interactive interface tutorial
- [Effective Go - Interfaces](https://go.dev/doc/effective_go#interfaces) — Interface design guidelines

---

> 📘 *This lesson is part of the [Go Programming](https://stanza.dev/courses/go) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*