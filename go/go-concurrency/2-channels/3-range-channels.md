---
source_course: "go-concurrency"
source_lesson: "go-concurrency-range-channels"
---

# Range Over Channels

## Introduction
Go's `for range` loop works with channels, providing a clean way to receive all values until the channel is closed. This pattern is the backbone of producer-consumer designs in Go.

## Key Concepts
- **Range Over Channel:** `for v := range ch` receives values from `ch` until it is closed and drained.
- **Channel Close as Signal:** Closing a channel tells all receivers that no more values will come.
- **Deadlock on Unclosed Channel:** If the channel is never closed, the range loop blocks forever.

## Real World Context
In a log processing pipeline, a producer goroutine reads log lines from a file and sends them to a channel. The consumer uses `for line := range logCh` to process each line. When the file is fully read, the producer closes the channel, and the consumer exits its loop cleanly.

## Deep Dive
You can use `for range` to iterate over values received from a channel. The loop terminates when the channel is closed and all buffered values are drained:

```go
ch := make(chan int, 2)
ch <- 1
ch <- 2
close(ch)

for v := range ch {
    fmt.Println(v)  // Prints 1, then 2, then exits
}
```

### Producer Pattern

```go
func generate(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)  // Always close!
        for _, n := range nums {
            out <- n
        }
    }()
    return out
}
```

The `defer close(out)` ensures the channel is closed even if the goroutine exits early due to a panic. Consumers using `for range` will receive all values and then exit the loop.

## Common Pitfalls
1. **Forgetting to close the channel** — If the producer never closes the channel, the consumer's `for range` loop blocks forever (deadlock or goroutine leak).
2. **Closing the channel before all values are sent** — This causes a panic on the next send. Always send all values first, then close.

## Best Practices
1. **Use `defer close(ch)` in the producer goroutine** — This guarantees the channel is closed even if the goroutine panics.
2. **Return receive-only channels from producers** — The generator pattern (`func gen() <-chan T`) encapsulates the goroutine and channel lifecycle.

## Summary
- `for v := range ch` receives until the channel is closed and drained.
- Always close the channel from the sender side using `defer close(ch)`.
- An unclosed channel causes the range loop to block forever.
- The generator pattern returns a receive-only channel and manages its lifecycle internally.

## Code Examples

**Producer-consumer pattern — the producer closes the channel when done, causing the range loop to exit**

```go
func producer(out chan<- int) {
    for i := 0; i < 5; i++ {
        out <- i
    }
    close(out) // Essential: signals consumers to stop
}

func main() {
    ch := make(chan int)
    go producer(ch)
    for n := range ch {
        fmt.Println(n)  // Prints 0, 1, 2, 3, 4 then exits
    }
}
```


## Resources

- [Go Tour: Range and Close](https://go.dev/tour/concurrency/4) — Interactive tutorial on ranging over channels and closing them

---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*