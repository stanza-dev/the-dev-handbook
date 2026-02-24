---
source_course: "go-concurrency"
source_lesson: "go-concurrency-atomic-operations"
---

# Atomic Operations

## Introduction
For simple counters and flags, the `sync/atomic` package provides lock-free operations that are faster than mutexes. Understanding when to use atomics versus mutexes is key to writing high-performance concurrent code.

## Key Concepts
- **Atomic Operation:** A CPU-level operation that completes in a single, indivisible step. No other goroutine can observe a partial state.
- **Compare-And-Swap (CAS):** Atomically sets a value only if it currently equals an expected value. The foundation of lock-free algorithms.
- **Atomic Types (Go 1.19+):** Type-safe wrappers like `atomic.Int64`, `atomic.Bool`, and `atomic.Pointer[T]`.

## Real World Context
A metrics counter in a high-throughput server (e.g., counting requests per second) should use `atomic.Int64` rather than a mutex-protected int. The atomic version avoids lock contention entirely, which matters when thousands of goroutines increment the counter simultaneously.

## Deep Dive
For simple counters or flags, `sync/atomic` is faster than `Mutex`:

```go
var ops atomic.Int64

// Increment safely
ops.Add(1)

// Read safely
val := ops.Load()

// Set value
ops.Store(100)

// Compare and swap
ops.CompareAndSwap(old, new)
```

### Atomic Types (Go 1.19+)

```go
var counter atomic.Int64
var flag atomic.Bool
var ptr atomic.Pointer[MyStruct]
```

### Legacy API (pre-1.19)

```go
var ops uint64
atomic.AddUint64(&ops, 1)
val := atomic.LoadUint64(&ops)
```

### When to Use Atomics
- Simple counters and flags.
- When mutex overhead is too high (verified by profiling).
- Building lock-free data structures (advanced).

**Warning:** Atomics are harder to use correctly than mutexes for anything beyond simple counters. When in doubt, use a mutex.

## Common Pitfalls
1. **Using `i++` and assuming it is atomic** — `i++` involves a read, modify, and write. Two goroutines doing `i++` simultaneously cause a data race. Use `atomic.Int64.Add(1)` instead.
2. **Mixing atomic and non-atomic access** — Every access to a shared variable must be atomic. Even a single non-atomic read creates a data race.

## Best Practices
1. **Prefer the typed API (Go 1.19+)** — `atomic.Int64` is safer and more readable than `atomic.AddInt64(&counter, 1)`.
2. **Use atomics only for simple values** — For compound operations (check-then-act), use a mutex.

## Summary
- `sync/atomic` provides lock-free operations for counters and flags.
- Go 1.19+ atomic types (`atomic.Int64`, `atomic.Bool`) are the preferred API.
- `i++` is NOT atomic — always use `atomic.Add` for shared counters.
- Use mutexes for compound operations; use atomics only for simple values.

## Code Examples

**Atomic counter using Go 1.19+ typed API — Add and Load are both lock-free and safe for concurrent use**

```go
import "sync/atomic"

var counter atomic.Int64

func increment() {
    counter.Add(1)
}

func getCount() int64 {
    return counter.Load()
}
```


## Resources

- [sync/atomic — Go Standard Library](https://pkg.go.dev/sync/atomic) — Official API reference for atomic operations and typed wrappers

---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*