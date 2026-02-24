---
source_course: "go-concurrency"
source_lesson: "go-concurrency-context-package"
---

# Understanding Context

## Introduction
The `context` package is Go's standard mechanism for carrying deadlines, cancellation signals, and request-scoped values across API boundaries. Every production Go service uses context extensively, making it one of the most important packages to master.

## Key Concepts
- **context.Context:** An interface carrying a deadline, cancellation signal, and key-value pairs across API boundaries.
- **context.Background():** The root context, used as the top of a context tree.
- **context.WithCancel / WithTimeout / WithDeadline:** Derived contexts that add cancellation capabilities to a parent context.

## Real World Context
In an HTTP handler, the request arrives with a context that is cancelled if the client disconnects. Every database query, external API call, and goroutine spawned to handle that request should use this context. If the client closes the connection, all downstream work is cancelled automatically, saving server resources.

## Deep Dive
Context carries deadlines, cancellation signals, and request-scoped values across API boundaries:

### Creating Contexts
- `context.Background()`: The root of all contexts.
- `context.TODO()`: Placeholder when unsure which context to use.
- `context.WithCancel(parent)`: Returns a copy that closes its Done channel when `cancel()` is called.
- `context.WithTimeout(parent, duration)`: Cancels automatically after duration.
- `context.WithDeadline(parent, time)`: Cancels at a specific time.

### Usage Pattern
Pass `ctx` as the first argument to functions:

```go
func operation(ctx context.Context) error {
    select {
    case <-time.After(5 * time.Second):
        return nil  // Work completed
    case <-ctx.Done():
        return ctx.Err()  // Cancelled or timed out
    }
}
```

### Context Hierarchy
Cancelling a parent cancels all derived child contexts:

```go
parent := context.Background()
ctx1, cancel1 := context.WithCancel(parent)
ctx2, _ := context.WithTimeout(ctx1, 10*time.Second)

cancel1() // Also cancels ctx2
```

## Common Pitfalls
1. **Forgetting to call the cancel function** — Every `WithCancel`, `WithTimeout`, and `WithDeadline` returns a cancel function. Failing to call it leaks the context's internal timer and prevents garbage collection of the child context.
2. **Storing context in a struct** — Contexts should flow through function parameters, not be stored in structs. Storing them breaks the compositional model.

## Best Practices
1. **Always `defer cancel()`** — Call cancel immediately after creating a derived context to ensure cleanup.
2. **Pass context as the first parameter named `ctx`** — This is a universally followed Go convention.

## Summary
- Context carries deadlines, cancellation, and values across API boundaries.
- Always call the cancel function (use `defer cancel()`).
- Cancelling a parent cancels all children.
- Pass context as the first parameter, never store it in a struct.

## Code Examples

**Timeout context — the 50ms deadline fires before the 1s timer, triggering ctx.Done() and printing the deadline error**

```go
ctx, cancel := context.WithTimeout(context.Background(), 50*time.Millisecond)
defer cancel() // Always call cancel to release resources

select {
case <-time.After(1 * time.Second):
    fmt.Println("overslept")
case <-ctx.Done():
    fmt.Println(ctx.Err()) // prints "context deadline exceeded"
}
```


## Resources

- [Go Blog: Go Concurrency Patterns: Context](https://go.dev/blog/context) — The original blog post introducing the context package and its design rationale
- [context package — Go Standard Library](https://pkg.go.dev/context) — Official API reference for the context package

---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*