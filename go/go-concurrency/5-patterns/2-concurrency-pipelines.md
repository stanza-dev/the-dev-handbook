---
source_course: "go-concurrency"
source_lesson: "go-concurrency-pipelines"
---

# Pipeline Pattern

## Introduction
A pipeline is a series of stages connected by channels, where each stage is a group of goroutines running the same function. Pipelines decompose complex processing into composable, concurrent stages.

## Key Concepts
- **Stage:** A function that receives values from an input channel, processes them, and sends results to an output channel.
- **Pipeline:** A chain of stages where the output of one stage feeds the input of the next.
- **Composability:** Stages are independent functions that can be recombined in different orders.

## Real World Context
A data processing pipeline might read records from a database (stage 1), transform them (stage 2), enrich them with data from an external API (stage 3), and write the results to a file (stage 4). Each stage runs concurrently, and the pipeline naturally handles backpressure through channel blocking.

## Deep Dive
A pipeline is a series of stages connected by channels:

```
Input -> [Stage 1] -> [Stage 2] -> [Stage 3] -> Output
```

### Example

```go
func gen(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for _, n := range nums {
            out <- n
        }
    }()
    return out
}

func square(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            out <- n * n
        }
    }()
    return out
}

// Usage: compose stages
for n := range square(gen(1, 2, 3, 4)) {
    fmt.Println(n)  // 1, 4, 9, 16
}
```

### Key Rules
- Each stage closes its output channel when done.
- Downstream stages read until the channel is closed.
- The pipeline is driven by the consumer pulling from the final stage.

## Common Pitfalls
1. **Not closing output channels** — If a stage forgets to close its output, the next stage blocks forever on `range`.
2. **No cancellation mechanism** — Without context or a done channel, a pipeline cannot be stopped early. Use `context.Context` to propagate cancellation through all stages.

## Best Practices
1. **Accept context in every stage** — Each stage should check `ctx.Done()` so the pipeline can be cancelled cleanly.
2. **Use `defer close(out)` in every stage goroutine** — This guarantees the output channel is closed even on panic.

## Summary
- A pipeline chains stages connected by channels.
- Each stage closes its output channel when done sending.
- Pipelines compose naturally: `square(filter(gen(...)))`.
- Always add context-based cancellation for production use.

## Code Examples

**Three-stage pipeline: generate numbers, filter evens, square — each stage runs concurrently in its own goroutine**

```go
func filter(in <-chan int, predicate func(int) bool) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            if predicate(n) {
                out <- n
            }
        }
    }()
    return out
}

// Pipeline: generate -> filter evens -> square
nums := gen(1, 2, 3, 4, 5, 6)
evens := filter(nums, func(n int) bool { return n%2 == 0 })
result := square(evens) // Produces: 4, 16, 36
```


## Resources

- [Go Blog: Go Concurrency Patterns: Pipelines and cancellation](https://go.dev/blog/pipelines) — The definitive guide to pipeline patterns with cancellation in Go

---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*