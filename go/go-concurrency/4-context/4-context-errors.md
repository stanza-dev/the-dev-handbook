---
source_course: "go-concurrency"
source_lesson: "go-concurrency-context-errors"
---

# Context Errors

## ctx.Err()

Returns the reason the context was cancelled:

*   `context.Canceled`: Explicitly cancelled via `cancel()`.
*   `context.DeadlineExceeded`: Timeout or deadline passed.

```go
if ctx.Err() == context.Canceled {
    log.Println("Request was cancelled by client")
} else if ctx.Err() == context.DeadlineExceeded {
    log.Println("Request timed out")
}
```

## Context Cause (Go 1.20+)

You can set a custom cause when cancelling:

```go
ctx, cancel := context.WithCancelCause(parent)

// Later
cancel(errors.New("user cancelled"))

// Check cause
cause := context.Cause(ctx)
```

## AfterFunc (Go 1.21+)

Register a function to run when context is done:

```go
stop := context.AfterFunc(ctx, func() {
    cleanup()
})
// Call stop() if you need to cancel the callback
```

## Code Examples

**Context Error Handling**

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


---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*