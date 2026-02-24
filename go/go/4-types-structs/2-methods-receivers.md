---
source_course: "go"
source_lesson: "go-methods-receivers"
---

# Methods & Receivers

## Introduction

Methods bring behavior to your types. In Go, a method is simply a function with a special receiver argument that associates it with a type. The choice between value and pointer receivers is one of the most important design decisions you will make, affecting both correctness and performance.

## Key Concepts

- **Receiver**: The parameter between `func` and the method name that binds the method to a type.
- **Value Receiver**: Receives a copy of the struct; cannot modify the original.
- **Pointer Receiver**: Receives a pointer to the struct; can modify the original.
- **Method Set**: The set of methods associated with a type, which determines interface satisfaction.

## Real World Context

In production Go code, the value-vs-pointer receiver decision impacts API safety and performance. HTTP handler structs, database repositories, and service layers almost always use pointer receivers because they hold mutable state or are too large to copy efficiently. Understanding method sets is essential when your types need to implement interfaces.

## Deep Dive

Methods are defined by placing a receiver between the `func` keyword and the method name.

```go
// Value receiver (receives a copy)
func (v Vertex) Abs() float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

// Pointer receiver (can modify the struct)
func (v *Vertex) Scale(f float64) {
    v.X = v.X * f
    v.Y = v.Y * f
}
```

A value receiver operates on a copy, so changes do not persist. A pointer receiver operates on the original value, so mutations are visible to the caller.

### When to Use Pointer Receivers

There are three primary reasons to choose a pointer receiver:

1. The method needs to modify the receiver.
2. The struct is large and copying would be expensive.
3. Consistency: if one method needs a pointer receiver, use pointer receivers for all methods on that type.

### Method Sets and Interface Satisfaction

A value `v` of type `T` can call both value and pointer receiver methods because Go automatically takes the address. A pointer `*T` can also call both. However, when implementing interfaces, the method set matters: a value of type `T` only includes value receiver methods, while `*T` includes both.

```go
type Sizer interface {
    Size() int
}

type File struct{ bytes int }

func (f *File) Size() int { return f.bytes }

var s Sizer = &File{100} // OK: *File has Size()
// var s Sizer = File{100} // Compile error: File doesn't have Size()
```

This distinction is critical when passing values to functions that accept interfaces.

### Methods on Any Named Type

You can define methods on any named type in your package, not just structs.

```go
type MyInt int

func (n MyInt) Double() MyInt {
    return n * 2
}
```

This is useful for adding domain-specific behavior to primitive types.

## Common Pitfalls

1. **Mixing receiver types inconsistently** — If some methods use pointer receivers and others use value receivers on the same type, the method set becomes confusing and interface satisfaction may fail unexpectedly.
2. **Expecting mutations from value receivers** — Modifying fields inside a value receiver method has no effect on the original struct, leading to silent bugs.

## Best Practices

1. **Be consistent with receiver types** — Pick one receiver type for a given struct and stick with it across all methods.
2. **Default to pointer receivers for structs** — Unless the struct is small and immutable, pointer receivers are safer and more performant.

## Summary

- Methods are functions with a receiver argument that binds them to a type.
- Value receivers operate on copies; pointer receivers operate on the original.
- Choose pointer receivers when you need mutation, large structs, or consistency.
- Method sets determine interface satisfaction: `*T` includes all methods, `T` only value receiver methods.
- You can define methods on any named type in your package, not just structs.

## Code Examples

**Struct Methods**

```go
type Rect struct {
    width, height int
}

func (r *Rect) Area() int {
    return r.width * r.height
}

func (r *Rect) Resize(w, h int) {
    r.width = w
    r.height = h
}

func main() {
    r := Rect{width: 10, height: 5}
    fmt.Println("Area:", r.Area())
    r.Resize(20, 10)
    fmt.Println("New Area:", r.Area())
}
```


## Resources

- [A Tour of Go - Methods](https://go.dev/tour/methods/1) — Interactive methods tutorial
- [Effective Go - Methods](https://go.dev/doc/effective_go#methods) — Method design guidelines

---

> 📘 *This lesson is part of the [Go Programming](https://stanza.dev/courses/go) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*