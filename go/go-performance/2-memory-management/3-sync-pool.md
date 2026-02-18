---
source_course: "go-performance"
source_lesson: "go-performance-sync-pool"
---

# Object Reuse

`sync.Pool` caches allocated objects for reuse, relieving pressure on the Garbage Collector. It is widely used for buffers (e.g., `bytes.Buffer`).

```go
var bufPool = sync.Pool{
    New: func() any {
        return new(bytes.Buffer)
    },
}

// Get from pool
b := bufPool.Get().(*bytes.Buffer)
b.Reset()  // Clear previous data!

// ... use buffer ...

// Put back
bufPool.Put(b)
```

## When to Use

*   High-frequency allocations of the same type.
*   Objects that are expensive to create.
*   Temporary buffers.

## Important Notes

*   Pools can be cleared by the runtime at any time (usually during GC).
*   Don't use pools for persistent state.
*   Always reset objects before returning to pool.

## Code Examples

**Pool Usage**

```go
var pool = sync.Pool{
    New: func() any {
        return make([]byte, 4096)
    },
}

func process(data []byte) {
    buf := pool.Get().([]byte)
    defer pool.Put(buf)
    
    // Use buf...
}
```


---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*