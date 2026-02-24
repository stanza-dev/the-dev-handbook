---
source_course: "go-std-lib"
source_lesson: "go-std-lib-tickers-timers"
---

# Tickers & Timers

## Introduction
Go's `time` package provides two channel-based scheduling primitives: timers that fire once and tickers that fire on a repeating interval. These are the building blocks for timeouts, periodic health checks, rate limiters, and background jobs. Understanding when to use each, and how to clean them up, prevents subtle goroutine leaks.

## Key Concepts
- **time.Timer**: Sends a single value on its channel `C` after a specified duration. Can be stopped or reset.
- **time.Ticker**: Sends a value on its channel `C` at regular intervals. Must be explicitly stopped.
- **time.After**: A convenience function that returns a channel for a one-shot delay. Cannot be stopped once created.
- **time.Tick**: A convenience function that returns a ticker channel but provides no way to stop it, so it leaks in most use cases.

## Real World Context
You are writing a worker that polls a job queue every 30 seconds but needs to shut down gracefully when the service receives a SIGTERM signal. A `time.NewTicker` drives the polling loop, and you combine it with a context cancellation in a `select` statement. When the context is cancelled, you call `ticker.Stop()` and exit. Without `Stop()`, the runtime would keep the ticker alive forever, leaking a goroutine per worker.

## Deep Dive

### Timers

A timer fires exactly once after the specified duration. You receive the event by reading from its channel `C`.

```go
timer := time.NewTimer(2 * time.Second)
<-timer.C
fmt.Println("Timer fired")
```

The timer sends the current time on `timer.C` after two seconds. This blocks the goroutine until the timer fires.

You can cancel a timer before it fires. If `Stop()` returns `false`, the timer already fired and you must drain the channel to avoid a stale value sitting in the buffer.

```go
if !timer.Stop() {
    <-timer.C // Drain the channel if already fired
}
```

Draining is necessary because Go's timer channels are buffered with a capacity of one. A lingering value can cause a subsequent `select` to read the old event.

### time.After

`time.After` is syntactic sugar for creating a one-shot timer and returning its channel. It is ideal for inline timeout patterns.

```go
select {
case <-time.After(1 * time.Second):
    fmt.Println("Timed out")
case result := <-ch:
    fmt.Println("Got result", result)
}
```

This `select` waits for either a result on `ch` or a one-second timeout, whichever comes first. Note that `time.After` allocates a new timer every call, so avoid using it in hot loops.

### Tickers

A ticker fires repeatedly at a fixed interval. Always stop it when you are done.

```go
ticker := time.NewTicker(1 * time.Second)
defer ticker.Stop()

for t := range ticker.C {
    fmt.Println("Tick at", t)
}
```

The `defer ticker.Stop()` line ensures the ticker's internal goroutine is released, even if the function returns early due to an error.

### time.Tick (Convenience)

`time.Tick` returns a ticker channel but gives you no handle to stop it. This means the underlying ticker runs forever.

```go
// Warning: Cannot be stopped, may leak!
for t := range time.Tick(1 * time.Second) {
    fmt.Println("Tick at", t)
}
```

Use `time.Tick` only in `main()` or top-level processes that run for the lifetime of the program. In all other cases, prefer `time.NewTicker` so you can call `Stop()`.

## Common Pitfalls
1. **Forgetting `ticker.Stop()`** — The runtime holds a reference to the ticker's internal channel, preventing garbage collection. Every unstopped ticker leaks a goroutine that sends values forever.
2. **Using `time.After` in a tight loop** — Each call allocates a new `time.Timer`. In a loop running thousands of times per second, this creates garbage pressure and wastes memory. Use `time.NewTimer` with `Reset()` instead.
3. **Not draining the timer channel after `Stop()`** — If `timer.Stop()` returns `false`, the value is already buffered. A subsequent `select` can pick up this stale event unless you drain it with `<-timer.C`.

## Best Practices
1. **Always `defer ticker.Stop()` immediately after `NewTicker`** — This guarantees cleanup regardless of how the function exits.
2. **Prefer `time.NewTimer` over `time.After` when you need cancellation** — `NewTimer` gives you a handle to call `Stop()` and `Reset()`, while `time.After` creates a fire-and-forget timer you cannot control.

## Summary
- `time.NewTimer` fires once and can be stopped or reset.
- `time.NewTicker` fires repeatedly and must be stopped with `Stop()` to avoid goroutine leaks.
- `time.After` is convenient for one-shot timeouts in `select` but allocates on every call.
- `time.Tick` leaks by design; use it only in long-lived top-level code.
- Always drain a timer channel after `Stop()` returns `false` to prevent stale reads.

## Code Examples

**Using time.After for a one-shot timeout in a select statement — the channel receives after the specified duration, enabling timeout patterns in concurrent code**

```go
select {
case <-time.After(1 * time.Second):
    fmt.Println("Timed out")
}
```


## Resources

- [time package - Ticker](https://pkg.go.dev/time#Ticker) — Official reference for time.Ticker and time.Timer
- [time package - Go Documentation](https://pkg.go.dev/time) — Official Go documentation for the time package

---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*