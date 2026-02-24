---
source_course: "go-performance"
source_lesson: "go-performance-false-sharing"
---

# False Sharing & Cache Contention

## Introduction
When two goroutines write to different variables that happen to share the same cache line, the CPU constantly invalidates and reloads that line between cores. This is called false sharing, and it can reduce parallel performance by 10x or more. Understanding and preventing false sharing is critical for lock-free concurrent data structures.

## Key Concepts
- **False sharing**: Two cores writing to different variables on the same 64-byte cache line, causing constant cache invalidation.
- **Cache line bouncing**: The cache line "bounces" between cores via the cache coherence protocol (MESI/MOESI), adding ~50-100 ns per write.
- **Cache line padding**: Adding unused bytes between variables to force them onto separate cache lines.
- **MESI protocol**: The cache coherence protocol that ensures all cores see consistent data. False sharing triggers its most expensive transition (Modified → Invalid).

## Real World Context
A concurrent counter partitioned across goroutines (each goroutine has its own counter) seems perfectly parallel — no locks, no shared state. But if the counters are adjacent in a struct or array, they share a cache line, and performance collapses. This bug has caused production performance issues at many companies.

## Deep Dive

### Demonstrating False Sharing

```go
type Counters struct {
    A int64  // Core 0 writes here
    B int64  // Core 1 writes here
    // A and B are adjacent — they share a 64-byte cache line!
}

var c Counters

// Two goroutines writing to "independent" counters
go func() { for i := 0; i < 1e8; i++ { atomic.AddInt64(&c.A, 1) } }()
go func() { for i := 0; i < 1e8; i++ { atomic.AddInt64(&c.B, 1) } }()
```

Despite no logical sharing, this is ~4x slower than running each goroutine alone because of cache line contention.

### Fixing with Cache Line Padding

```go
type PaddedCounters struct {
    A   int64
    _   [56]byte  // Padding: 64 - 8 = 56 bytes
    B   int64
    _   [56]byte
}
```

Now A and B are on separate cache lines. Each core can write independently without invalidating the other's cache.

### atomic.Int64 Does NOT Pad

A common misconception: `sync/atomic.Int64` does NOT include cache line padding. It is only 8 bytes (plus a `noCopy` guard). If you need padded atomic counters, you must add padding yourself:

```go
type PaddedAtomic struct {
    value atomic.Int64
    _     [56]byte  // Must add padding manually
}
```

### When False Sharing Matters

False sharing is significant when:
- Multiple cores write frequently to adjacent memory
- The data structure is in a hot loop (millions of operations)
- Lock-free algorithms using atomic operations

It does NOT matter when:
- Only one goroutine accesses the data
- Writes are infrequent (< 1000/sec)
- The data is read-only (multiple readers don't cause cache invalidation)

## Common Pitfalls
1. **Assuming atomic types are padded** — `atomic.Int64`, `atomic.Uint64`, etc. are NOT cache-line padded. You must add padding manually for concurrent write-heavy workloads.
2. **Over-padding** — Adding padding to every struct wastes memory. Only pad when you've confirmed false sharing via benchmarks.

## Best Practices
1. **Benchmark before padding** — Use `go test -bench -cpu=1,2,4,8` to detect false sharing. If performance drops with more cores, false sharing is likely.
2. **Pad only write-heavy concurrent fields** — Read-only data does not cause cache invalidation, so padding it wastes memory.

## Summary
- False sharing occurs when goroutines write to different variables on the same 64-byte cache line.
- It causes cache line bouncing between cores, adding ~50-100 ns per write.
- Fix by adding `[56]byte` padding between concurrently-written fields.
- `atomic.Int64` is NOT cache-line padded — manual padding is required for high-contention scenarios.
- Only pad write-heavy fields confirmed as bottlenecks via benchmarking.

## Code Examples

**Fixing false sharing with cache line padding — atomic.Int64 is only 8 bytes, so 56 bytes of padding fills the 64-byte cache line**

```go
// False sharing: A and B share a cache line
type BadCounters struct {
	A atomic.Int64  // 8 bytes — same cache line as B
	B atomic.Int64  // 8 bytes
}

// Fixed: each counter on its own cache line
type GoodCounters struct {
	A atomic.Int64
	_ [56]byte       // Pad to 64-byte cache line boundary
	B atomic.Int64
	_ [56]byte
}

// Benchmark with: go test -bench=. -cpu=1,2,4,8
// Bad: performance drops with more cores
// Good: scales linearly with cores
```


## Resources

- [sync/atomic Package](https://pkg.go.dev/sync/atomic) — API reference for Go's atomic operations — note the types are NOT cache-line padded

---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*