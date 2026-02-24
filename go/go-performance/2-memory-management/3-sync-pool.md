---
source_course: "go-performance"
source_lesson: "go-performance-sync-pool"
---

# sync.Pool for Object Reuse

## Introduction
When a hot path allocates and discards the same type of object millions of times, the garbage collector does enormous work tracking and freeing those objects. `sync.Pool` lets you reuse objects instead of allocating new ones, dramatically reducing GC pressure.

## Key Concepts
- **sync.Pool**: A concurrent-safe pool of temporary objects that can be reused across goroutines.
- **Pool lifecycle**: Objects may be silently removed from the pool during any GC cycle. Never rely on an object being in the pool.
- **New function**: The `Pool.New` function creates a fresh object when the pool is empty.
- **Get/Put**: `Get()` retrieves an object (or calls New), `Put()` returns it to the pool.

## Real World Context
The standard library itself uses sync.Pool extensively. `fmt.Fprintf` pools its internal buffers. `encoding/json` pools its encoders. The `net/http` package pools bufio.Reader/Writer instances. These pools reduce allocations by 50-90% in high-throughput servers.

## Deep Dive

### Basic Usage Pattern

```go
var bufferPool = sync.Pool{
    New: func() any {
        return new(bytes.Buffer)
    },
}

func processRequest(data []byte) string {
    // Get a buffer from the pool (or create a new one)
    buf := bufferPool.Get().(*bytes.Buffer)

    // IMPORTANT: Reset the buffer before use
    buf.Reset()

    // Use the buffer
    buf.Write(data)
    result := buf.String()

    // Return the buffer to the pool for reuse
    bufferPool.Put(buf)

    return result
}
```

### When sync.Pool Helps

sync.Pool is most effective when:
- The object is **frequently allocated and discarded** (high allocation rate)
- The object is **expensive to create** (large buffers, complex initialization)
- **Multiple goroutines** compete for the same type of allocation

### When sync.Pool Does NOT Help

- One-time or rare allocations (pool overhead outweighs savings)
- Very small objects (the pool's internal overhead may exceed the allocation cost)
- Objects that must persist across GC cycles (pool contents are cleared during GC)

## Common Pitfalls
1. **Forgetting to reset objects** — A pooled buffer still contains data from its previous use. Always call `Reset()` or zero out fields after `Get()`.
2. **Storing state in pooled objects** — Pool objects may be cleared by GC at any time. Never use a pool as a cache.
3. **Pooling tiny objects** — A pool for `int` or small structs adds overhead without benefit. Pool large buffers and complex objects.

## Best Practices
1. **Benchmark before and after** — Use `go test -bench -benchmem` to verify the pool actually reduces allocations.
2. **Reset before use, not before put** — Resetting on Get is safer because Put might not always be called (panics, early returns).

## Summary
- sync.Pool reuses objects across goroutines, reducing heap allocations.
- Objects in the pool may be silently cleared during GC — never rely on persistence.
- Always reset pooled objects before reuse to avoid data leaks.
- Most effective for large, frequently allocated objects in high-throughput paths.
- The standard library uses sync.Pool extensively (fmt, encoding/json, net/http).

## Code Examples

**Pooling byte slices in an HTTP handler — each request reuses a buffer instead of allocating a new one**

```go
var bufPool = sync.Pool{
	New: func() any {
		return make([]byte, 0, 4096) // Pre-allocate 4KB capacity
	},
}

func handleRequest(w http.ResponseWriter, r *http.Request) {
	// Get a buffer from pool — avoids heap allocation
	buf := bufPool.Get().([]byte)
	buf = buf[:0] // Reset length, keep capacity

	// Use the buffer...
	buf = append(buf, "response data"...)
	w.Write(buf)

	// Return to pool for reuse by another goroutine
	bufPool.Put(buf)
}
```


## Resources

- [sync.Pool Package](https://pkg.go.dev/sync#Pool) — Official API reference for sync.Pool including usage guidelines

---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*