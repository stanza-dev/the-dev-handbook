---
source_course: "go-concurrency"
source_lesson: "go-concurrency-fan-out-fan-in"
---

# Fan-Out / Fan-In

## Fan-Out

Multiple goroutines reading from the same channel to parallelize work:

```go
jobs := gen(1, 2, 3, 4, 5)

// Fan-out: Multiple workers process jobs
worker1 := process(jobs)
worker2 := process(jobs)
worker3 := process(jobs)
```

## Fan-In

Merging multiple channels into a single channel:

```go
func merge(cs ...<-chan int) <-chan int {
    out := make(chan int)
    var wg sync.WaitGroup
    
    for _, c := range cs {
        wg.Add(1)
        go func(ch <-chan int) {
            defer wg.Done()
            for n := range ch {
                out <- n
            }
        }(c)
    }
    
    go func() {
        wg.Wait()
        close(out)
    }()
    
    return out
}

// Usage
result := merge(worker1, worker2, worker3)
```

## Combined Pattern

Fan-out to parallelize CPU-intensive work, then fan-in to collect results.

## Code Examples

**Fan-In with Select**

```go
func fanIn(ch1, ch2 <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for ch1 != nil || ch2 != nil {
            select {
            case v, ok := <-ch1:
                if !ok {
                    ch1 = nil
                } else {
                    out <- v
                }
            case v, ok := <-ch2:
                if !ok {
                    ch2 = nil
                } else {
                    out <- v
                }
            }
        }
    }()
    return out
}
```


---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*