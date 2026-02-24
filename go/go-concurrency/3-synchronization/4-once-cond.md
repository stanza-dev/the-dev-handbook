---
source_course: "go-concurrency"
source_lesson: "go-concurrency-once-cond"
---

# Once & Cond

## Introduction
Beyond Mutex and WaitGroup, the `sync` package offers specialized primitives: `Once` for one-time initialization and `Cond` for condition-based signaling between goroutines.

## Key Concepts
- **sync.Once:** Ensures a function runs exactly once, even when called from multiple goroutines concurrently. All callers block until the function completes.
- **sync.Cond:** A condition variable that lets goroutines wait for and signal arbitrary conditions. Built on top of a Locker (usually a Mutex).

## Real World Context
`sync.Once` is the standard way to implement lazy singleton initialization in Go — database connection pools, configuration loading, and logger setup all use this pattern. `sync.Cond` is less common (channels are usually preferred) but appears in low-level libraries like database drivers that need to signal when a connection becomes available.

## Deep Dive

### sync.Once
Ensures a function runs exactly once, even with concurrent calls:

```go
var once sync.Once
var instance *Singleton

func GetInstance() *Singleton {
    once.Do(func() {
        instance = &Singleton{}
        instance.init()
    })
    return instance
}
```

Use cases: lazy initialization, singleton pattern, one-time setup.

### sync.Cond (Condition Variable)
For signaling between goroutines. Less common than channels but useful for broadcast notifications:

```go
var mu sync.Mutex
cond := sync.NewCond(&mu)

// Waiting goroutine
cond.L.Lock()
for !condition {
    cond.Wait()  // Atomically unlocks and waits
}
// condition is true, we have the lock
cond.L.Unlock()

// Signaling goroutine
cond.L.Lock()
condition = true
cond.Signal()     // Wake one waiter
// or cond.Broadcast()  // Wake all waiters
cond.L.Unlock()
```

## Common Pitfalls
1. **Checking condition without a loop** — Spurious wakeups can occur. Always use `for !condition { cond.Wait() }`, never `if !condition`.
2. **Using Once.Do with a function that panics** — If the function passed to `Do` panics, `Once` still considers it "done" and will not retry on subsequent calls.

## Best Practices
1. **Prefer channels over sync.Cond** — Channels are simpler and compose better. Use `Cond` only when you need to broadcast to multiple waiters.
2. **Use `sync.OnceValue` (Go 1.21+)** — For initializing and returning a value, `sync.OnceValue` is cleaner than `Once` with a package-level variable.

## Summary
- `sync.Once` guarantees exactly-one execution, ideal for lazy initialization.
- `sync.Cond` provides Wait/Signal/Broadcast for condition-based coordination.
- Always use a `for` loop (not `if`) around `cond.Wait()` to handle spurious wakeups.
- Prefer channels over `sync.Cond` unless you need broadcast semantics.

## Code Examples

**Lazy configuration loading with sync.Once — the config file is read exactly once, even if getConfig is called from multiple goroutines**

```go
var loadConfigOnce sync.Once
var config *Config

func getConfig() *Config {
    loadConfigOnce.Do(func() {
        data, _ := os.ReadFile("config.json")
        config = &Config{}
        json.Unmarshal(data, config)
    })
    return config
}
```


## Resources

- [sync.Once — Go Standard Library](https://pkg.go.dev/sync#Once) — API reference for Once, OnceFunc, OnceValue, and OnceValues

---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*