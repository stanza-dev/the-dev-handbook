---
source_course: "go-concurrency"
source_lesson: "go-concurrency-worker-pool"
---

# Worker Pools

## Introduction
Unbounded goroutine creation can overwhelm downstream resources like databases and APIs. Worker pools limit concurrency by processing jobs through a fixed number of goroutines, providing natural backpressure.

## Key Concepts
- **Worker Pool:** A fixed number of goroutines that read from a shared jobs channel and write to a results channel.
- **Backpressure:** When all workers are busy, new job senders block until a worker is free, naturally throttling throughput.
- **Bounded Concurrency:** Unlike spawning a goroutine per task, worker pools cap the number of concurrent operations.

## Real World Context
A thumbnail generation service that needs to process uploaded images should not spawn 10,000 goroutines for 10,000 uploads — that would exhaust memory and overwhelm the disk. A worker pool of, say, 20 workers processes images in batches, keeping resource usage predictable.

## Deep Dive
To limit concurrency (e.g., only 5 database connections at once), use a Worker Pool:

### Pattern
1. Create a `jobs` channel and a `results` channel.
2. Spawn `N` worker goroutines that loop over the `jobs` channel.
3. Send tasks to `jobs`.
4. Close `jobs` to signal no more work.

```go
func worker(id int, jobs <-chan int, results chan<- int) {
    for j := range jobs {
        fmt.Println("worker", id, "processing", j)
        results <- j * 2
    }
}
```

### Benefits
- Limits concurrent operations to N.
- Reuses goroutines instead of creating new ones per task.
- Backpressure: if workers are busy, job senders block.

## Common Pitfalls
1. **Not closing the jobs channel** — If you forget to close the jobs channel, all workers block forever on `range jobs` after the last job, causing a goroutine leak.
2. **Reading results before all jobs are sent** — If the results channel is unbuffered and you try to read results before sending all jobs, you can deadlock. Either buffer the results channel or use a separate goroutine to collect results.

## Best Practices
1. **Size the pool based on the bottleneck** — If workers hit a database, size the pool to match the connection pool. If workers are CPU-bound, size to `runtime.NumCPU()`.
2. **Use errgroup for error-aware worker pools** — `errgroup.Group` with `SetLimit()` provides a cleaner API with automatic error handling.

## Summary
- Worker pools limit concurrency via a fixed number of goroutines reading from a jobs channel.
- Close the jobs channel to signal completion; workers exit their range loop.
- Backpressure naturally throttles senders when all workers are busy.
- Consider `errgroup.SetLimit()` as a higher-level alternative.

## Code Examples

**Worker pool with 3 workers processing 5 jobs — close(jobs) signals workers to exit after draining the queue**

```go
func main() {
    jobs := make(chan int, 100)
    results := make(chan int, 100)

    // Start 3 workers
    for w := 1; w <= 3; w++ {
        go worker(w, jobs, results)
    }

    // Send 5 jobs
    for j := 1; j <= 5; j++ {
        jobs <- j
    }
    close(jobs)

    // Collect results
    for a := 1; a <= 5; a++ {
        <-results
    }
}
```


## Resources

- [Go by Example: Worker Pools](https://gobyexample.com/worker-pools) — Step-by-step worker pool implementation with explanations

---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*