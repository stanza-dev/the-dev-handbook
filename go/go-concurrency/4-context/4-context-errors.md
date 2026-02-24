---
source_course: "go-concurrency"
source_lesson: "go-concurrency-context-errors"
---

# Context Errors & Causes

## Introduction
When a context is cancelled, `ctx.Err()` tells you why. Go 1.20 added `context.Cause` for custom error causes, and Go 1.21 added `context.AfterFunc` for cleanup callbacks. These features let you build nuanced error handling around cancellation.

## Key Concepts
- **context.Canceled:** The error returned when a context is explicitly cancelled via `cancel()`.
- **context.DeadlineExceeded:** The error returned when a context's timeout or deadline has passed.
- **context.Cause (Go 1.20+):** Returns the custom error passed to `CancelCauseFunc`, providing richer cancellation reasons.

## Real World Context
In an API gateway, distinguishing between `Canceled` (the client disconnected) and `DeadlineExceeded` (the backend took too long) determines the HTTP response code: 499 Client Closed Request vs. 504 Gateway Timeout. Custom causes via `WithCancelCause` can carry even more specific reasons.

## Deep Dive

### ctx.Err()
Returns the reason the context was cancelled:

- `context.Canceled`: Explicitly cancelled via `cancel()`.
- `context.DeadlineExceeded`: Timeout or deadline passed.

```go
if ctx.Err() == context.Canceled {
    log.Println("Request was cancelled by client")
} else if ctx.Err() == context.DeadlineExceeded {
    log.Println("Request timed out")
}
```

### Context Cause (Go 1.20+)
You can set a custom cause when cancelling:

```go
ctx, cancel := context.WithCancelCause(parent)

// Later
cancel(errors.New("user cancelled"))

// Check cause
cause := context.Cause(ctx)
```

### AfterFunc (Go 1.21+)
Register a function to run when context is done:

```go
stop := context.AfterFunc(ctx, func() {
    cleanup()
})
// Call stop() to cancel the callback if context is not yet done
```

## Common Pitfalls
1. **Checking ctx.Err() before ctx.Done()** — `ctx.Err()` returns nil until the context is actually cancelled. Check `<-ctx.Done()` first, then inspect `ctx.Err()`.
2. **Ignoring the cause in error messages** — When using `WithCancelCause`, always check `context.Cause(ctx)` for the most specific error, not just `ctx.Err()`.

## Best Practices
1. **Use `WithCancelCause` for debuggable cancellation** — Custom causes make it easier to trace why a context was cancelled in logs and error messages.
2. **Use `AfterFunc` for cleanup** — It replaces manual goroutine-plus-select patterns for running cleanup when a context expires.

## Summary
- `ctx.Err()` returns `context.Canceled` or `context.DeadlineExceeded`.
- `context.Cause` (Go 1.20+) provides custom cancellation reasons.
- `context.AfterFunc` (Go 1.21+) registers cleanup callbacks on context cancellation.
- Always check `ctx.Done()` before inspecting `ctx.Err()`.

## Code Examples

**Differentiating cancellation from timeout — the handler returns different errors based on why the context was cancelled**

```go
func handleRequest(ctx context.Context) error {
    result, err := doWork(ctx)
    if err != nil {
        if ctx.Err() == context.Canceled {
            return errors.New("client disconnected")
        }
        if ctx.Err() == context.DeadlineExceeded {
            return errors.New("request timeout")
        }
        return err
    }
    return nil
}
```


## Resources

- [context package — Go Standard Library](https://pkg.go.dev/context) — API reference including WithCancelCause, Cause, and AfterFunc

---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*