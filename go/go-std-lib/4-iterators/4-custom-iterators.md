---
source_course: "go-std-lib"
source_lesson: "go-std-lib-custom-iterators"
---

# Custom Iterators

## Introduction
The standard library gives you `slices.Values`, `maps.Keys`, and other ready-made iterators, but the real power of Go's iterator pattern comes when you write your own. Custom iterators like Filter, Map, and Take let you build composable, lazy data pipelines that read like declarative queries but execute with zero unnecessary allocations.

## Key Concepts
- **Filter**: An iterator that yields only the elements from a source iterator that satisfy a predicate function.
- **Map (Transform)**: An iterator that applies a transformation function to each element, yielding the transformed values.
- **Take**: An iterator that yields at most `n` elements from a source iterator, then stops.
- **Chaining**: Nesting iterator constructors so data flows through a pipeline of transformations lazily.

## Real World Context
You are building an analytics dashboard that processes a stream of events. You need to filter for purchase events, transform each into a revenue amount, and take the first 100 for a preview table. With custom iterators, this becomes `slices.Collect(Take(Map(Filter(events, isPurchase), toRevenue), 100))`. Each step is lazy: if the 100th purchase event comes after only scanning 200 events out of millions, the rest are never touched.

## Deep Dive

### Filter

A Filter iterator wraps a source iterator and only yields elements that pass a predicate.

```go
func Filter[V any](seq iter.Seq[V], predicate func(V) bool) iter.Seq[V] {
    return func(yield func(V) bool) {
        for v := range seq {
            if predicate(v) {
                if !yield(v) {
                    return
                }
            }
        }
    }
}
```

The outer function returns a new `iter.Seq[V]`. Inside, it ranges over the source `seq`, checks the predicate, and only calls `yield` for matching elements. The `!yield(v)` check respects early termination.

### Map (Transform)

A Map iterator applies a function to each element, changing its type or value.

```go
func Map[V, R any](seq iter.Seq[V], transform func(V) R) iter.Seq[R] {
    return func(yield func(R) bool) {
        for v := range seq {
            if !yield(transform(v)) {
                return
            }
        }
    }
}
```

Notice the type parameters: `V` is the input type, `R` is the output type. This lets you transform `iter.Seq[User]` into `iter.Seq[string]` by extracting names, for example.

### Take

A Take iterator limits how many elements flow through.

```go
func Take[V any](seq iter.Seq[V], n int) iter.Seq[V] {
    return func(yield func(V) bool) {
        count := 0
        for v := range seq {
            if count >= n {
                return
            }
            if !yield(v) {
                return
            }
            count++
        }
    }
}
```

Once `count` reaches `n`, the iterator returns, which stops ranging over the source. This is critical for infinite sequences: without Take, an infinite iterator would run forever.

### Chaining Iterators

The real power emerges when you nest these constructors. Each one wraps the previous, forming a lazy pipeline.

```go
result := slices.Collect(
    Take(
        Map(
            Filter(slices.Values(nums), isEven),
            double,
        ),
        3,
    ),
)
```

This reads inside-out: start with `nums`, filter to even numbers, double each one, take the first 3, then collect into a slice. Each step pulls values from the previous step on demand. No intermediate slices are allocated.

## Common Pitfalls
1. **Forgetting to check `!yield(v)` in every custom iterator** — If your Map or Filter ignores yield's return value, a downstream `break` does not propagate, and the pipeline keeps running uselessly.
2. **Nesting too deeply without readability aids** — Deeply nested `Take(Map(Filter(...)))` chains become hard to read. Consider assigning intermediate iterators to named variables for clarity.
3. **Mutating shared state inside a transform function** — If your Map function closes over and mutates external state, the pipeline is no longer safe to reuse or run concurrently.

## Best Practices
1. **Keep iterator combinators generic** — Use type parameters (`[V any]`) so your Filter, Map, and Take work with any element type. This maximizes reuse.
2. **Test custom iterators with both complete consumption and early break** — Verify your iterator works when the consumer reads all values and when the consumer breaks after the first element. Both paths must be correct.

## Summary
- Filter, Map, and Take are the foundational custom iterator patterns in Go.
- Each wraps a source `iter.Seq` and returns a new `iter.Seq`, enabling composable lazy pipelines.
- Always check `yield`'s return value in every custom iterator to respect early termination.
- Chaining iterators avoids intermediate allocations: data flows on demand, one element at a time.
- Use generics to make your iterator combinators reusable across all types.

## Code Examples

**Infinite Iterator**

```go
// Generate infinite sequence
func Naturals() iter.Seq[int] {
    return func(yield func(int) bool) {
        for i := 1; ; i++ {
            if !yield(i) {
                return
            }
        }
    }
}

// Usage with Take
for n := range Take(Naturals(), 5) {
    fmt.Println(n)  // 1, 2, 3, 4, 5
}
```


## Resources

- [iter package - Go Documentation](https://pkg.go.dev/iter) — Official reference for building custom iterators with iter.Seq and iter.Seq2

---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*