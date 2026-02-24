---
source_course: "go-std-lib"
source_lesson: "go-std-lib-iter-slices-maps"
---

# The iter Package

## Introduction
Go 1.23 did not just add range-over-function syntax; it also wired iterator support into the standard library's `slices` and `maps` packages. These built-in iterator functions let you convert between collections and lazy sequences, compose pipelines, and collect results without writing boilerplate loops. As of Go 1.26, even the `reflect` package exposes iterators like `Type.Fields()` and `Type.Methods()`, confirming that the pattern is here to stay.

## Key Concepts
- **slices.Values**: Returns an `iter.Seq2[int, E]` that yields each element of a slice in order.
- **slices.Backward**: Returns an `iter.Seq2[int, E]` that yields index-value pairs in reverse order.
- **slices.Collect**: Materializes an `iter.Seq2[int, E]` into a concrete `[]V` slice.
- **maps.Keys / maps.Values / maps.All**: Return iterators over a map's keys, values, or key-value pairs respectively.

## Real World Context
You have a map of user IDs to scores and you need a sorted leaderboard. Before iterators, you would manually collect keys into a slice, sort it, and loop again. With iterator functions, you write `for id := range maps.Keys(scores)` to get the keys as a lazy sequence, collect them, sort, and you are done. The code reads like a pipeline instead of imperative bookkeeping.

## Deep Dive

### slices Package

The `slices` package provides functions to create iterators from slices and to materialize iterators back into slices.

```go
import "slices"

for v := range slices.Values(mySlice) {
    fmt.Println(v)
}
```

`slices.Values` wraps a slice into an `iter.Seq2[int, E]` that yields each element. The iteration is lazy: elements are produced one at a time as the loop requests them.

You can also iterate in reverse.

```go
for _, v := range slices.Backward(mySlice) {
    fmt.Println(v)
}
```

`slices.Backward` yields elements from last to first. This is useful for processing stacks, undo histories, or any data where you need reverse-chronological order.

To materialize an iterator back into a concrete slice, use `slices.Collect`.

```go
result := slices.Collect(myIterator)
```

This consumes the entire iterator and returns a new `[]V` slice. Use it at the end of a pipeline when you need a concrete collection.

### maps Package

The `maps` package provides three iterator functions for maps.

```go
import "maps"

for k := range maps.Keys(myMap) {
    fmt.Println(k)
}

for v := range maps.Values(myMap) {
    fmt.Println(v)
}

for k, v := range maps.All(myMap) {
    fmt.Printf("%s: %v\n", k, v)
}
```

`maps.Keys` returns an `iter.Seq[K]`, `maps.Values` returns an `iter.Seq2[int, E]`, and `maps.All` returns an `iter.Seq2[K, V]`. Iteration order is randomized, just like ranging over a map directly.

### Round-Tripping Between Slices and Iterators

You can convert freely between slices and iterators.

```go
seq := slices.Values([]int{1, 2, 3})
slice := slices.Collect(seq)
```

`Values` creates the lazy iterator; `Collect` eagerly materializes it. This round-trip is the foundation for building filter/map/take pipelines that end with `Collect`.

## Common Pitfalls
1. **Calling `slices.Collect` on an infinite iterator** — If the iterator never stops yielding, `Collect` will run forever and exhaust memory. Always use `Take` or a bounded iterator before collecting.
2. **Expecting deterministic order from `maps.Keys`** — Map iteration in Go is intentionally randomized. If you need sorted keys, collect them into a slice first and sort.
3. **Reusing a collected iterator** — `slices.Collect` consumes the iterator. If you need to iterate again, create a new iterator from the original collection or re-materialize.

## Best Practices
1. **Use `slices.Values` as the entry point for pipelines** — Convert your slice to an iterator, chain transformations (Filter, Map, Take), then call `slices.Collect` to materialize the result.
2. **Prefer `maps.All` when you need both keys and values** — It avoids the overhead of two separate iterations and keeps your loop body simple with `for k, v := range maps.All(m)`.

## Summary
- The `slices` and `maps` packages provide built-in iterator functions as of Go 1.23.
- `slices.Values` and `slices.Backward` create lazy iterators from slices; `slices.Collect` materializes iterators back into slices.
- `maps.Keys`, `maps.Values`, and `maps.All` create iterators over map contents.
- The iterator pattern continues to expand across the standard library, with Go 1.26's `reflect` package adding `Type.Fields()` and `Type.Methods()` iterators.
- Always bound infinite iterators before calling `Collect` to avoid unbounded memory growth.

## Code Examples

**Filtering with Iterators**

```go
import (
    "slices"
    "maps"
)

func main() {
    nums := []int{1, 2, 3, 4, 5}
    
    // Filter and collect
    evens := slices.Collect(
        Filter(slices.Values(nums), func(n int) bool {
            return n%2 == 0
        }),
    )
    fmt.Println(evens) // [2 4]
}
```


## Resources

- [iter package - Go Documentation](https://pkg.go.dev/iter) — Official Go documentation for the iter package
- [slices package - Go Documentation](https://pkg.go.dev/slices) — Official reference for iterator-based slice functions like slices.All and slices.Collect

---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*