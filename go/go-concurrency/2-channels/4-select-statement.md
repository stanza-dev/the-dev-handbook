---
source_course: "go-concurrency"
source_lesson: "go-concurrency-select-statement"
---

# The Select Statement

## Introduction
The `select` statement is Go's way of waiting on multiple channel operations simultaneously. It is to channels what `switch` is to values — but with the added power of non-deterministic choice when multiple cases are ready.

## Key Concepts
- **Select:** A control structure that blocks until one of its channel operations can proceed.
- **Random Selection:** If multiple cases are ready simultaneously, one is chosen at random.
- **Default Case:** If present, executes immediately when no other case is ready, making the select non-blocking.

## Real World Context
In an HTTP server with graceful shutdown, a `select` statement waits on both the server error channel and an OS signal channel. Whichever fires first determines whether the server crashed or was asked to shut down, and the appropriate cleanup runs.

## Deep Dive
`select` lets a goroutine wait on multiple channel operations:

```go
select {
case v := <-ch1:
    fmt.Println("From ch1:", v)
case v := <-ch2:
    fmt.Println("From ch2:", v)
case ch3 <- 42:
    fmt.Println("Sent to ch3")
default:
    fmt.Println("No communication ready")
}
```

### Rules
- If multiple cases are ready, one is chosen at random.
- `default` executes immediately if no other case is ready.
- Without `default`, select blocks until a case is ready.

### Common Patterns

#### Timeout
```go
select {
case v := <-ch:
    process(v)
case <-time.After(1 * time.Second):
    fmt.Println("Timeout")
}
```

#### Non-blocking Send/Receive
```go
select {
case ch <- v:
    fmt.Println("Sent")
default:
    fmt.Println("Channel full, dropping value")
}
```

## Common Pitfalls
1. **Using `time.After` in a loop** — Each call to `time.After` creates a new timer that is not garbage collected until it fires. In a tight loop, this leaks memory. Use `time.NewTimer` with `Reset()` instead.
2. **Forgetting the `default` case in a polling loop** — Without `default`, the select blocks. If you need non-blocking behavior, always include `default`.

## Best Practices
1. **Prefer `context.Context` over `time.After` for timeouts** — Contexts compose and propagate cancellation through call chains.
2. **Use select to multiplex done channels** — Combine `ctx.Done()` with work channels to make goroutines cancellation-aware.

## Summary
- `select` waits on multiple channel operations; if multiple are ready, one is chosen at random.
- `default` makes select non-blocking.
- Use `select` with `ctx.Done()` for cancellation-aware goroutines.
- Avoid `time.After` in loops; use `time.NewTimer` with `Reset()` instead.

## Code Examples

**Select multiplexing two channels — the faster channel's case fires first, then the slower one**

```go
func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)
    
    go func() {
        time.Sleep(100 * time.Millisecond)
        ch1 <- "one"
    }()
    go func() {
        time.Sleep(200 * time.Millisecond)
        ch2 <- "two"
    }()
    
    for i := 0; i < 2; i++ {
        select {
        case msg := <-ch1:
            fmt.Println(msg)
        case msg := <-ch2:
            fmt.Println(msg)
        }
    }
}
```


## Resources

- [Go Tour: Select](https://go.dev/tour/concurrency/5) — Interactive tutorial on the select statement and its behavior

---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*