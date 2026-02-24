---
source_course: "go-std-lib"
source_lesson: "go-std-lib-pull-iterators"
---

# Pull-Based Iteration

## Introduction
Go's standard iterators are push-based: the iterator drives the loop by calling `yield` for each value. But some algorithms need the consumer to be in control, pulling one value at a time from multiple sources. The `iter.Pull` function bridges this gap by converting any push iterator into a `next`/`stop` pair, giving you manual control over when and how values are consumed.

## Key Concepts
- **Push iterator**: The standard `func(yield func(V) bool)` pattern where the iterator drives the loop and pushes values to the consumer.
- **Pull iterator**: A `next()` function that the consumer calls to get values one at a time, plus a `stop()` function for cleanup.
- **iter.Pull**: Converts a push `iter.Seq[V]` into a pull-style `(next func() (V, bool), stop func())` pair.
- **iter.Pull2**: The `Seq2` variant that converts `iter.Seq2[K, V]` into `(next func() (K, V, bool), stop func())`.

## Real World Context
You are building a merge-sort style algorithm that reads from two sorted streams and produces a single sorted output. With push iterators, you cannot independently advance one stream without advancing the other. With `iter.Pull`, you convert both streams into pull iterators, peek at the head of each, and advance only the one with the smaller value. This pattern applies to any merge, zip, or interleave operation across multiple data sources.

## Deep Dive
`iter.Pull` takes a push iterator and returns two functions: `next` and `stop`.

```go
next, stop := iter.Pull(mySeq)
defer stop() // Always call stop!

for {
    v, ok := next()
    if !ok {
        break
    }
    fmt.Println(v)
}
```

Calling `next()` returns the next value and a boolean indicating whether a value was available. When `ok` is `false`, the iterator is exhausted. The `defer stop()` ensures cleanup runs regardless of how the loop exits.

Under the hood, `iter.Pull` runs the push iterator in a separate goroutine that suspends after each `yield` call, waiting for the consumer to call `next()`. This goroutine-based implementation is what makes `stop()` essential: without it, the goroutine hangs forever, leaking resources.

The classic use case for pull iterators is zipping two sequences together. A push iterator cannot do this because it controls the loop, and you cannot interleave two push loops. With Pull, you advance each source independently.

```go
func Zip(seq1, seq2 iter.Seq[int]) iter.Seq2[int, int] {
    return func(yield func(int, int) bool) {
        next1, stop1 := iter.Pull(seq1)
        defer stop1()
        next2, stop2 := iter.Pull(seq2)
        defer stop2()

        for {
            v1, ok1 := next1()
            v2, ok2 := next2()
            if !ok1 || !ok2 {
                return
            }
            if !yield(v1, v2) {
                return
            }
        }
    }
}
```

This `Zip` function itself returns a push iterator (`iter.Seq2`), but internally uses Pull to advance two sources in lockstep. The deferred `stop1()` and `stop2()` calls ensure both goroutines are cleaned up.

Other use cases include interleaving values from multiple channels, implementing merge joins on sorted data, and building stateful consumers that need to pause and resume iteration at arbitrary points.

## Common Pitfalls
1. **Forgetting to call `stop()`** — `iter.Pull` spawns a goroutine internally. If you never call `stop()`, that goroutine blocks forever, leaking memory and a goroutine slot. Always `defer stop()` immediately after calling `iter.Pull`.
2. **Calling `next()` after `stop()`** — Once you call `stop()`, the underlying goroutine terminates. Subsequent calls to `next()` will return the zero value and `false`, but relying on this is fragile and unclear. Treat `stop()` as final.
3. **Using Pull when a push iterator would suffice** — Pull adds goroutine overhead. If a simple `for v := range seq` works, use it. Reserve Pull for cases where you truly need manual control over multiple iterators.

## Best Practices
1. **Always `defer stop()` on the same line as `iter.Pull`** — This makes the cleanup obligation visible and ensures it runs even on early returns or panics.
2. **Prefer push iterators when possible** — Pull introduces goroutine synchronization overhead. Only convert to pull style when you need to interleave, zip, or manually advance multiple iterators.

## Summary
- Push iterators drive the loop; pull iterators let the consumer drive.
- `iter.Pull` converts a push `iter.Seq` into a `next`/`stop` function pair by running the iterator in a goroutine.
- Always call `stop()` (via `defer`) to avoid goroutine leaks.
- Use Pull for zipping, merging, or interleaving multiple iterators where independent advancement is required.
- Prefer push iterators for simple, single-source iteration to avoid goroutine overhead.

## Code Examples

**Zipping two sequences**

```go
func Zip(seq1, seq2 iter.Seq[int]) iter.Seq2[int, int] {
    return func(yield func(int, int) bool) {
        next1, stop1 := iter.Pull(seq1)
        defer stop1()
        next2, stop2 := iter.Pull(seq2)
        defer stop2()

        for {
            v1, ok1 := next1()
            v2, ok2 := next2()
            if !ok1 || !ok2 {
                return
            }
            if !yield(v1, v2) {
                return
            }
        }
    }
}
```


## Resources

- [iter.Pull - Go Documentation](https://pkg.go.dev/iter#Pull) — Official reference for iter.Pull and iter.Pull2 pull-based iteration

---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*