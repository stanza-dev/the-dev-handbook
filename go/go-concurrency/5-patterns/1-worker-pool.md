---
source_course: "go-concurrency"
source_lesson: "go-concurrency-worker-pool"
---

# Worker Pools

To limit concurrency (e.g., only 5 database connections at once), use a Worker Pool.

## Pattern
1.  Create a `jobs` channel and a `results` channel.
2.  Spawn `N` worker goroutines that loop over the `jobs` channel.
3.  Send tasks to `jobs`.
4.  Close `jobs` to signal no more work.

```go
func worker(id int, jobs <-chan int, results chan<- int) {
    for j := range jobs {
        fmt.Println("worker", id, "processing", j)
        results <- j * 2
    }
}
```

## Benefits

*   Limits concurrent operations.
*   Reuses goroutines instead of creating new ones.
*   Backpressure: If workers are busy, job senders will block.

## Code Examples

**Worker Pool Implementation**

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


---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*