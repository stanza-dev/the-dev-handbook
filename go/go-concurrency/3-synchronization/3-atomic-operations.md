---
source_course: "go-concurrency"
source_lesson: "go-concurrency-atomic-operations"
---

# Low-Level Sync

For simple counters or flags, `sync/atomic` is faster than `Mutex`.

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

## Atomic Types (Go 1.19+)

```go
var counter atomic.Int64
var flag atomic.Bool
var ptr atomic.Pointer[MyStruct]
```

## Legacy API

```go
var ops uint64
atomic.AddUint64(&ops, 1)
val := atomic.LoadUint64(&ops)
```

## When to Use

*   Simple counters and flags.
*   When mutex overhead is too high.
*   Building lock-free data structures (advanced).

**Warning:** Atomics are harder to use correctly than mutexes.

## Code Examples

**Atomic Counter**

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


---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*