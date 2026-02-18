---
source_course: "go-concurrency"
source_lesson: "go-concurrency-once-cond"
---

# sync.Once

Ensures a function runs exactly once, even with concurrent calls.

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

## Use Cases

*   Lazy initialization.
*   Singleton pattern.
*   One-time setup.

# sync.Cond (Condition Variable)

For signaling between goroutines. Less common than channels.

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
cond.Signal()  // Wake one waiter
// or cond.Broadcast()  // Wake all waiters
cond.L.Unlock()
```

## Code Examples

**Lazy Configuration**

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


---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*