---
source_course: "go-concurrency"
source_lesson: "go-concurrency-waitgroup-mutex"
---

# sync.WaitGroup
Use `WaitGroup` to wait for a collection of goroutines to finish.

1.  `Add(delta int)`: Increment counter.
2.  `Done()`: Decrement counter (usually deferred).
3.  `Wait()`: Block until counter is zero.

**Common Bug:** Passing `WaitGroup` by value. Always pass by pointer or use closure capture.

# sync.Mutex
Use `Mutex` to protect shared memory (critical sections).

```go
var mu sync.Mutex

mu.Lock()
defer mu.Unlock()
// access shared resource
```

See [The Go Memory Model](https://go.dev/ref/mem).

## Code Examples

**Correct WaitGroup Usage**

```go
var wg sync.WaitGroup

for i := 0; i < 3; i++ {
    wg.Add(1)
    go func(i int) {
        defer wg.Done()
        fmt.Println(i)
    }(i)
}

wg.Wait()
```


---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*