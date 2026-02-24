---
source_course: "go-concurrency"
source_lesson: "go-concurrency-fan-out-fan-in"
---

# Fan-Out / Fan-In

## Introduction
Fan-out distributes work from one channel to multiple goroutines; fan-in merges results from multiple channels into one. Together they form a powerful pattern for parallelizing CPU-intensive or I/O-bound work.

## Key Concepts
- **Fan-Out:** Multiple goroutines reading from the same channel, each processing a subset of the work.
- **Fan-In:** A merge function that reads from multiple input channels and sends all values to a single output channel.
- **Combined Pattern:** Fan-out to parallelize work, fan-in to collect results.

## Real World Context
A search engine query fans out to multiple index shards (each searched by a separate goroutine), then fans in the results from all shards into a single ranked list. This pattern reduces latency by parallelizing I/O-bound searches across shards.

## Deep Dive

### Fan-Out
Multiple goroutines reading from the same channel to parallelize work:

```go
jobs := gen(1, 2, 3, 4, 5)

// Fan-out: Multiple workers process jobs from the same channel
worker1 := process(jobs)
worker2 := process(jobs)
worker3 := process(jobs)
```

### Fan-In
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

The WaitGroup ensures the output channel is closed only after all input channels are drained.

## Common Pitfalls
1. **Forgetting to close the merged output channel** — Without the WaitGroup-and-close goroutine, the consumer of the merged channel blocks forever.
2. **Not handling channel closure in fan-in select** — When using `select` instead of range, you must handle the `ok == false` case and set the channel to `nil` to disable that select case.

## Best Practices
1. **Use WaitGroup in the merge function** — It cleanly tracks when all input channels are drained, ensuring the output channel is closed exactly once.
2. **Add context cancellation** — Pass `ctx` to both fan-out workers and the merge function so the entire fan-out/fan-in can be cancelled.

## Summary
- Fan-out: multiple goroutines read from one channel to parallelize work.
- Fan-in: merge multiple channels into one using a WaitGroup to track completion.
- Always close the merged output channel after all inputs are drained.
- Add context cancellation for production use.

## Code Examples

**Fan-in with select — setting a closed channel to nil disables its select case, preventing busy-looping on zero values**

```go
func fanIn(ch1, ch2 <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for ch1 != nil || ch2 != nil {
            select {
            case v, ok := <-ch1:
                if !ok {
                    ch1 = nil  // Disable this case
                } else {
                    out <- v
                }
            case v, ok := <-ch2:
                if !ok {
                    ch2 = nil  // Disable this case
                } else {
                    out <- v
                }
            }
        }
    }()
    return out
}
```


## Resources

- [Go Blog: Go Concurrency Patterns](https://go.dev/blog/pipelines) — Official blog post covering fan-out, fan-in, and pipeline cancellation

---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*