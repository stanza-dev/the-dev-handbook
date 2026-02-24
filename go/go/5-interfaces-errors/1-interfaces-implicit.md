---
source_course: "go"
source_lesson: "go-interfaces-implicit"
---

# Implicit Interfaces

## Introduction

Interfaces are one of Go's most powerful features, and they work differently from most languages. In Go, a type implements an interface simply by having the right methods. There is no `implements` keyword, no explicit declaration. This implicit satisfaction enables remarkable decoupling between packages.

## Key Concepts

- **Interface**: A set of method signatures that define a behavior contract.
- **Implicit Satisfaction**: A type implements an interface automatically if it has all the required methods.
- **Interface Value**: A two-word data structure holding a concrete type and a concrete value.
- **Nil Interface Pitfall**: An interface is nil only when both its type and value components are nil.

## Real World Context

Implicit interfaces are what make Go's standard library so composable. The `io.Reader` and `io.Writer` interfaces are implemented by files, network connections, buffers, compressors, and more, all without those types knowing about each other. In application code, implicit interfaces enable dependency injection and testability: define a small interface where you need it and any matching type satisfies it.

## Deep Dive

An interface declares a set of method signatures. Any type whose methods match the signatures implements the interface automatically.

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

No registration step is needed. The compiler verifies satisfaction at the point of use.

### Interface Values

An interface value holds a concrete type and a value of that type. You can assign any satisfying value to it.

```go
var s Stringer
s = Person{"Alice"}
fmt.Println(s.String())  // Alice
```

Under the hood, `s` stores both the type descriptor (`Person`) and the data (`{"Alice"}`).

### The Nil Interface Trap

An interface value is nil only when both its type and value components are nil. This is a common source of bugs.

```go
var s Stringer      // nil interface (type=nil, value=nil)
var p *Person       // nil pointer
s = p              // s is NOT nil! (type=*Person, value=nil)
```

Even though `p` is nil, assigning it to an interface sets the type component, making the interface non-nil. Always check interface nilness directly rather than checking the underlying pointer.

## Common Pitfalls

1. **The nil interface vs nil concrete value** — Assigning a nil pointer to an interface makes the interface non-nil, causing unexpected behavior in `if err != nil` checks.
2. **Overly large interfaces** — Defining interfaces with many methods reduces the number of types that can satisfy them. Prefer small, focused interfaces.

## Best Practices

1. **Define interfaces where they are used, not where they are implemented** — This keeps packages decoupled and interfaces minimal.
2. **Keep interfaces small** — The Go proverb says: "The bigger the interface, the weaker the abstraction." One or two methods is ideal.

## Summary

- Go interfaces are satisfied implicitly: no `implements` keyword is needed.
- An interface value stores both a concrete type and a concrete value.
- A nil pointer assigned to an interface makes the interface non-nil, which is a common trap.
- Define small interfaces where they are consumed for maximum flexibility.
- Implicit satisfaction enables powerful decoupling and testability.

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