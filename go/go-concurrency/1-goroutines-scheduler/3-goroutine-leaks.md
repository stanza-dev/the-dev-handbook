---
source_course: "go-concurrency"
source_lesson: "go-concurrency-goroutine-leaks"
---

# Preventing Goroutine Leaks

## Introduction
A goroutine leak occurs when a goroutine is started but never terminates. Leaked goroutines silently consume memory and can eventually crash a long-running service. Learning to prevent them is essential for production Go code.

## Key Concepts
- **Goroutine Leak:** A goroutine that is blocked forever or running an infinite loop without an exit path.
- **Blocked Send/Receive:** A channel operation that will never complete because there is no corresponding receiver or sender.
- **Nil Channel:** Sending to or receiving from a `nil` channel blocks forever.

## Real World Context
In a microservice handling thousands of requests per second, even a single leaked goroutine per request means tens of thousands of leaked goroutines per hour. Monitoring `runtime.NumGoroutine()` in production dashboards is a standard practice to catch leaks before they cause OOM kills.

## Deep Dive

### Common Causes
1. **Blocked Send:** Sending to a channel that has no receiver.
2. **Blocked Receive:** Waiting on a channel that will never send or close.
3. **Nil Channel:** Sending or receiving from a nil channel blocks forever.
4. **Infinite Loop:** A loop without exit condition or proper cancellation.

Detect leaks by monitoring the goroutine count:

```go
fmt.Println("Active goroutines:", runtime.NumGoroutine())
```

### Solutions
- Always ensure there is a path for the goroutine to exit.
- Use `context.Context` for cancellation.
- Close channels when done sending.
- Use buffered channels when a sender should not block.

Here is a common leak scenario and its fix:

```go
// LEAK: nothing ever sends to ch
func leak() {
    ch := make(chan int)
    go func() {
        val := <-ch // Blocks forever
        fmt.Println(val)
    }()
}

// FIXED: use context for cancellation
func noLeak(ctx context.Context) {
    ch := make(chan int)
    go func() {
        select {
        case val := <-ch:
            fmt.Println(val)
        case <-ctx.Done():
            return
        }
    }()
}
```

## Common Pitfalls
1. **Forgetting to close channels** — If a producer goroutine exits without closing its output channel, consumers block forever waiting for more data.
2. **Not using context for cancellation** — Without a cancellation mechanism, background goroutines have no way to know when they should stop.

## Best Practices
1. **Monitor goroutine count in production** — Export `runtime.NumGoroutine()` as a metric and alert on unexpected growth.
2. **Use `goleak` in tests** — The `go.uber.org/goleak` package detects goroutine leaks in unit tests.

## Summary
- Goroutine leaks consume memory and can crash long-running services.
- Common causes: blocked channel operations, nil channels, infinite loops.
- Always provide an exit path via `context.Context` or channel closing.
- Monitor `runtime.NumGoroutine()` and use `goleak` in tests.

## Code Examples

**A goroutine leak caused by receiving on a channel that never gets a value — the goroutine blocks forever**

```go
func leak() {
    ch := make(chan int)
    go func() {
        val := <-ch // Blocks forever because nothing sends to ch
        fmt.Println(val)
    }()
}
// The anonymous goroutine is leaked — it will never terminate.
```


## Resources

- [goleak — Goroutine Leak Detector](https://pkg.go.dev/go.uber.org/goleak) — Uber's library for detecting goroutine leaks in tests

---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*