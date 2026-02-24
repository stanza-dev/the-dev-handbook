---
source_course: "go-std-lib"
source_lesson: "go-std-lib-iter-basics"
---

# Range Over Functions

## Introduction
Go 1.23 introduced a powerful new capability: you can now use `range` over functions, not just slices, maps, and channels. This lets you write custom iterators that plug directly into `for` loops, enabling lazy evaluation, infinite sequences, and composable data pipelines. Since their introduction, iterators have become a well-established part of the Go ecosystem, with the `reflect` package in Go 1.26 adding iterator methods like `Type.Fields()` and `Type.Methods()` as a sign of broad adoption.

## Key Concepts
- **Range over functions**: A `for` loop can iterate over any function whose signature matches `func(yield func(V) bool)` (single value) or `func(yield func(K, V) bool)` (key-value pair).
- **yield**: A callback provided by the runtime that represents the loop body. Calling `yield(v)` sends a value to the loop. If yield returns `false`, the loop has terminated (e.g., via `break`).
- **iter.Seq[V]**: A type alias defined in the `iter` package for single-value iterator functions: `func(yield func(V) bool)`.
- **iter.Seq2[K, V]**: A type alias for key-value iterator functions: `func(yield func(K, V) bool)`.

## Real World Context
Consider a database cursor that streams rows one at a time. Without iterators, you would need to expose a `Next()/Value()` API or load all rows into a slice. With range-over-function, you write an iterator that yields rows lazily, and consumers use a clean `for row := range db.QueryRows(ctx, sql) { ... }` loop. The caller does not need to know whether rows come from memory, disk, or the network.

## Deep Dive
The core pattern is a function that accepts a `yield` callback and calls it for each element. The function returns when iteration is complete.

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
```

The `Countdown` function returns an iterator. When used in a `for` loop, Go calls this function and passes the loop body as the `yield` callback.

Using the iterator is as clean as ranging over a slice.

```go
for x := range Countdown(5) {
    fmt.Println(x) // 5, 4, 3, 2, 1
}
```

Each call to `yield(i)` executes the loop body with `x = i`. If the loop body contains a `break`, `yield` returns `false`, and the iterator should return immediately.

The `yield` function is the bridge between your iterator logic and the `for` loop. When `yield` returns `true`, the loop wants more values. When it returns `false`, the loop has exited (via `break`, `return`, or `goto`), and your iterator must stop producing values. Ignoring a `false` return wastes resources and violates the iterator contract.

The `iter` package provides two type aliases that make iterator signatures cleaner and more self-documenting.

```go
type Seq[V any]      func(yield func(V) bool)
type Seq2[K, V any]  func(yield func(K, V) bool)
```

Using these types, the `Countdown` function signature becomes `func Countdown(n int) iter.Seq[int]`, which immediately communicates that it returns an iterator of integers.

## Common Pitfalls
1. **Ignoring the return value of `yield`** — If you continue calling `yield` after it returns `false`, you violate the iterator contract. The Go runtime may panic in debug builds, and your iterator wastes CPU cycles producing values nobody consumes.
2. **Confusing the function shape with a regular callback** — An iterator function is called once by the `for` loop, and `yield` is called many times inside it. It is not a callback that runs per-element from outside.
3. **Forgetting that iterators are lazy** — No work happens until someone ranges over the iterator. If you build a chain of Filter/Map/Take but never range or Collect, nothing executes.

## Best Practices
1. **Always check `yield`'s return value** — The idiomatic pattern is `if !yield(v) { return }`. This single line respects early termination and keeps your iterator correct.
2. **Use `iter.Seq` and `iter.Seq2` in public APIs** — The type aliases are self-documenting and signal to callers that a function returns an iterator they can range over.

## Summary
- Go 1.23 introduced range-over-function iterators, now deeply integrated across the standard library (including `reflect` in Go 1.26).
- An iterator is a function matching `func(yield func(V) bool)` that calls `yield` for each value.
- `yield` returns `false` when the consumer stops; always check it and return immediately.
- `iter.Seq[V]` and `iter.Seq2[K, V]` are the standard type aliases for iterator signatures.
- Iterators enable lazy evaluation: no work happens until someone ranges over them.

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


## Resources

- [Go 1.23 Release Notes - Range over function iterators](https://go.dev/doc/go1.23#language) — Official release notes explaining range-over-func iterators introduced in Go 1.23
- [iter package - Go Documentation](https://pkg.go.dev/iter) — Official reference for the iter package with Seq and Seq2 types

---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*