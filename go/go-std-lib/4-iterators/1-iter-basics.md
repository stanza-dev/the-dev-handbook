---
source_course: "go-std-lib"
source_lesson: "go-std-lib-iter-basics"
---

# Go 1.23 Iterators

Go 1.23 introduces standard support for custom iterators via the `range` keyword over functions.

## The Pattern
A function with signature `func(yield func(V) bool)` can be used in a `for` loop.

```go
func Countdown(n int) func(yield func(int) bool) {
    return func(yield func(int) bool) {
        for i := n; i > 0; i-- {
            if !yield(i) {
                return // Stop iteration
            }
        }
    }
}

// Usage
for x := range Countdown(5) {
    fmt.Println(x)  // 5, 4, 3, 2, 1
}
```

## Understanding yield

The `yield` function is provided by the runtime (the body of the `for` loop). If `yield` returns `false`, it means the loop broke (e.g., via `break`), and your iterator should stop.

## iter.Seq Type

The `iter` package defines type aliases:

```go
type Seq[V any] func(yield func(V) bool)
type Seq2[K, V any] func(yield func(K, V) bool)
```

## Code Examples

**Key-Value Iterator**

```go
import "iter"

// Seq2 yields key-value pairs
func Enumerate[V any](s []V) iter.Seq2[int, V] {
    return func(yield func(int, V) bool) {
        for i, v := range s {
            if !yield(i, v) {
                return
            }
        }
    }
}
```


---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*