---
source_course: "go-architecture"
source_lesson: "go-architecture-generics-basics"
---

# Generics (Go 1.18+)

Generics allow you to write functions and data structures that work with different types while maintaining type safety.

```go
func Map[T any, R any](s []T, f func(T) R) []R {
    result := make([]R, len(s))
    for i, v := range s {
        result[i] = f(v)
    }
    return result
}
```

## Type Parameters

*   `[T any]`: T can be any type.
*   `[T comparable]`: T must support `==` and `!=`.
*   `[T constraints.Ordered]`: T must support `<`, `<=`, `>`, `>=`.

## Basic Constraints

```go
// any: alias for interface{}
func Print[T any](v T) { fmt.Println(v) }

// comparable: supports == and !=
func Contains[T comparable](s []T, v T) bool {
    for _, x := range s {
        if x == v { return true }
    }
    return false
}
```

## Code Examples

**Ordered Constraint**

```go
import "golang.org/x/exp/constraints"

func Min[T constraints.Ordered](x, y T) T {
    if x < y {
        return x
    }
    return y
}
```


---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*