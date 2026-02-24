---
source_course: "go-concurrency"
source_lesson: "go-concurrency-rwmutex"
---

# Read-Write Mutex

## Introduction
When reads vastly outnumber writes, `sync.RWMutex` provides better throughput than a regular `Mutex` by allowing multiple concurrent readers while still giving writers exclusive access.

## Key Concepts
- **sync.RWMutex:** A reader/writer mutual exclusion lock. Multiple goroutines can hold the read lock simultaneously, but only one can hold the write lock.
- **RLock/RUnlock:** Acquire and release a shared read lock.
- **Lock/Unlock:** Acquire and release an exclusive write lock.

## Real World Context
A configuration cache read thousands of times per second but updated only on config file changes is a perfect use case. Using `Mutex` would serialize all reads unnecessarily. `RWMutex` lets all readers proceed in parallel, only blocking when a writer needs to update the config.

## Deep Dive
When reads are much more frequent than writes, `RWMutex` provides better performance:

- Multiple readers can hold the lock simultaneously.
- Writers get exclusive access — no readers or other writers can hold the lock.

```go
var rwmu sync.RWMutex
var data map[string]string

func read(key string) string {
    rwmu.RLock()
    defer rwmu.RUnlock()
    return data[key]
}

func write(key, value string) {
    rwmu.Lock()
    defer rwmu.Unlock()
    data[key] = value
}
```

### When to Use
- Many concurrent readers, few writers.
- Read operations significantly outnumber writes.
- Read operations do not need to see immediately updated values.

### Writer Priority
Go's `RWMutex` gives pending writers priority over new readers. Once a writer calls `Lock()`, new `RLock()` calls block until the writer finishes. This prevents writer starvation but means a burst of write requests can temporarily block readers.

## Common Pitfalls
1. **Upgrading RLock to Lock** — You cannot upgrade a read lock to a write lock. Calling `Lock()` while holding `RLock()` causes a deadlock. Release the read lock first.
2. **Using RWMutex when writes are frequent** — If writes are common, the overhead of RWMutex exceeds that of a plain Mutex. Profile before choosing.

## Best Practices
1. **Profile before switching from Mutex to RWMutex** — The benefit depends on the read/write ratio. For write-heavy workloads, plain Mutex is simpler and often faster.
2. **Keep read-locked sections short** — Long read-locked sections delay writers, increasing latency for updates.

## Summary
- `RWMutex` allows concurrent readers but exclusive writers.
- Go's implementation gives pending writers priority over new readers, preventing writer starvation.
- Never try to upgrade RLock to Lock — it causes deadlock.
- Profile to confirm the read/write ratio justifies RWMutex over Mutex.

## Code Examples

**Thread-safe cache using RWMutex — Get uses RLock for concurrent reads, Set uses Lock for exclusive writes**

```go
type SafeCache struct {
    mu    sync.RWMutex
    items map[string]Item
}

func (c *SafeCache) Get(key string) (Item, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    item, ok := c.items[key]
    return item, ok
}

func (c *SafeCache) Set(key string, item Item) {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.items[key] = item
}
```


## Resources

- [sync.RWMutex — Go Standard Library](https://pkg.go.dev/sync#RWMutex) — API reference for RWMutex with detailed behavior documentation

---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*