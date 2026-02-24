---
source_course: "go-architecture"
source_lesson: "go-architecture-generic-structs"
---

# Generic Data Structures

## Introduction
Generics are not just for functions. You can define generic structs, creating reusable data structures like stacks, sets, and result types that work with any element type while maintaining full type safety. This eliminates the need for code generation or `interface{}` containers.

## Key Concepts
- **Generic Struct**: A struct with type parameters (e.g., `Stack[T any]`) that is instantiated with specific types.
- **Method Receivers**: Methods on generic types must include the type parameter in the receiver (e.g., `func (s *Stack[T]) Push(item T)`).
- **Zero Value**: The `var zero T` idiom creates the zero value for any type parameter.

## Real World Context
Before generics, a thread-safe set required either code generation (go generate) or storing `interface{}` values and performing type assertions everywhere. With generics, you write `Set[string]` and get compile-time safety.

## Deep Dive
A generic stack demonstrates the core pattern:

```go
type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(item T) {
    s.items = append(s.items, item)
}

func (s *Stack[T]) Pop() (T, bool) {
    if len(s.items) == 0 {
        var zero T
        return zero, false
    }
    item := s.items[len(s.items)-1]
    s.items = s.items[:len(s.items)-1]
    return item, true
}
```

You instantiate generic types with explicit or inferred type arguments:

```go
s := Stack[int]{}   // Explicit type
s.Push(10)
s.Push(20)
val, ok := s.Pop()  // val = 20, ok = true
```

A generic `Result` type can replace the error tuple pattern:

```go
type Result[T any] struct {
    Value T
    Error error
}

func Ok[T any](v T) Result[T] {
    return Result[T]{Value: v}
}

func Err[T any](err error) Result[T] {
    return Result[T]{Error: err}
}
```

## Common Pitfalls
1. **Omitting the type parameter in method receivers** — You must write `func (s *Stack[T])`, not `func (s *Stack)`. The compiler requires the type parameter.
2. **Trying to add new type parameters to methods** — Go does not allow methods to introduce additional type parameters beyond those on the type definition.

## Best Practices
1. **Use the zero value idiom** — `var zero T` is the idiomatic way to get the zero value of a type parameter.
2. **Constrain only when needed** — Use `any` for containers. Use `comparable` if you need map keys or equality checks.

## Summary
- Generic structs let you build type-safe, reusable data structures.
- Method receivers must include the type parameter from the type definition.
- The `var zero T` idiom produces the zero value for any type.
- You cannot add new type parameters to methods—only to the type itself.

## Code Examples

**A generic Set implementation using comparable constraint — the empty struct value minimizes memory usage while the type parameter ensures compile-time safety**

```go
// A generic Set backed by a map. comparable constraint
// is required because map keys must support ==.
type Set[T comparable] struct {
    items map[T]struct{}
}

func NewSet[T comparable]() *Set[T] {
    return &Set[T]{items: make(map[T]struct{})}
}

func (s *Set[T]) Add(item T) {
    s.items[item] = struct{}{}
}

func (s *Set[T]) Contains(item T) bool {
    _, ok := s.items[item]
    return ok
}

// Usage: s := NewSet[string](); s.Add("hello")
```


## Resources

- [Go Blog — An Introduction To Generics](https://go.dev/blog/intro-generics) — Official blog post covering generic types, functions, and constraints

---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*