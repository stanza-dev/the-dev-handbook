---
source_course: "go-architecture"
source_lesson: "go-architecture-generics-best-practices"
---

# When to Use Generics

## Good Use Cases

*   **Data structures:** Stacks, Queues, Trees, Sets.
*   **Utility functions:** Map, Filter, Reduce.
*   **Algorithms:** Sort, Search.

## When NOT to Use

*   **Simple types:** If concrete type works, use it.
*   **Few types:** If only 2-3 types, separate functions might be clearer.
*   **Interface behavior:** If behavior varies, use interfaces.

## Guidelines

```go
// Bad: Unnecessary generic
func ToString[T any](v T) string {
    return fmt.Sprintf("%v", v)
}

// Good: Just use any
func ToString(v any) string {
    return fmt.Sprintf("%v", v)
}
```

## Type Inference

Let the compiler infer types when possible:

```go
// Explicit (verbose)
result := Map[int, string](nums, strconv.Itoa)

// Inferred (clean)
result := Map(nums, strconv.Itoa)
```

## Zero Values

```go
func Zero[T any]() T {
    var zero T
    return zero
}
```

## Code Examples

**Filter Function**

```go
// Good generic use case: utility functions
func Filter[T any](s []T, f func(T) bool) []T {
    result := make([]T, 0)
    for _, v := range s {
        if f(v) {
            result = append(result, v)
        }
    }
    return result
}

// Usage
evens := Filter(nums, func(n int) bool { return n%2 == 0 })
```


---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*