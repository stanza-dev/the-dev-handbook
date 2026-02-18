---
source_course: "go-concurrency"
source_lesson: "go-concurrency-rwmutex"
---

# sync.RWMutex

When reads are much more frequent than writes, `RWMutex` provides better performance.

*   Multiple readers can hold the lock simultaneously.
*   Writers get exclusive access.

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

## When to Use

*   Many concurrent readers, few writers.
*   Read operations significantly outnumber writes.
*   Read operations don't need to see immediately updated values.

## Caution

*   Don't upgrade RLock to Lock (deadlock!).
*   Writers can starve if there are many readers.

## Code Examples

**RWMutex Cache**

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


---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*