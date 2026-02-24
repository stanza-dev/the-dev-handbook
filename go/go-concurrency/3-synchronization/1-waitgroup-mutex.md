---
source_course: "go-concurrency"
source_lesson: "go-concurrency-waitgroup-mutex"
---

# Mutex & WaitGroup

## Introduction
While channels are Go's preferred communication mechanism, sometimes you need to protect shared memory directly. The `sync` package provides `Mutex` for mutual exclusion and `WaitGroup` for waiting on goroutine completion.

## Key Concepts
- **sync.Mutex:** Provides exclusive access to a shared resource. Only one goroutine can hold the lock at a time.
- **sync.WaitGroup:** A counter that blocks until it reaches zero. Used to wait for a collection of goroutines to finish.
- **defer mu.Unlock():** The idiomatic way to release a lock, ensuring it is released even if the function panics.

## Real World Context
In a concurrent web scraper, a `WaitGroup` tracks how many pages are still being fetched. A `Mutex` protects the shared results map where each goroutine stores its scraped data. Without the mutex, concurrent map writes would cause a runtime panic (Go maps are not safe for concurrent use).

## Deep Dive

### sync.WaitGroup
Use `WaitGroup` to wait for a collection of goroutines to finish:

1. `Add(delta int)`: Increment counter before starting a goroutine.
2. `Done()`: Decrement counter (usually deferred inside the goroutine).
3. `Wait()`: Block until counter is zero.

**Common Bug:** Passing `WaitGroup` by value. Always pass by pointer or use closure capture.

### sync.Mutex
Use `Mutex` to protect shared memory (critical sections):

```go
var mu sync.Mutex

mu.Lock()
defer mu.Unlock()
// access shared resource safely
```

See [The Go Memory Model](https://go.dev/ref/mem) for the formal guarantees.

## Common Pitfalls
1. **Passing WaitGroup by value** — `sync.WaitGroup` contains internal state. Passing it by value creates a copy, and `Done()` on the copy does not affect the original. Always pass by pointer.
2. **Calling `wg.Add` inside the goroutine** — The goroutine might not start before `Wait()` is called, leading to a race. Always call `Add` before the `go` statement.

## Best Practices
1. **Use `defer mu.Unlock()` immediately after `Lock()`** — This ensures the lock is released even on early returns or panics.
2. **Keep critical sections short** — Hold the lock for the minimum time needed to reduce contention.

## Summary
- `sync.WaitGroup`: Add before starting, Done inside (deferred), Wait to block.
- `sync.Mutex`: Lock/Unlock for exclusive access to shared data.
- Always pass WaitGroup by pointer and call Add before the goroutine.
- Use `defer mu.Unlock()` immediately after locking.

## Code Examples

**WaitGroup pattern — Add before the goroutine, defer Done inside, Wait after all are launched**

```go
var wg sync.WaitGroup

for i := 0; i < 3; i++ {
    wg.Add(1)
    go func(i int) {
        defer wg.Done()
        fmt.Println(i)
    }(i)
}

wg.Wait() // Blocks until all 3 goroutines call Done()
```


## Resources

- [sync package — Go Standard Library](https://pkg.go.dev/sync) — Official API reference for Mutex, WaitGroup, and other sync primitives
- [The Go Memory Model](https://go.dev/ref/mem) — Formal specification of happens-before guarantees in Go

---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*