---
source_course: "go-concurrency"
source_lesson: "go-concurrency-context-package"
---

# The Context Package

Context carries deadlines, cancellation signals, and request-scoped values across API boundaries and between processes.

## Creating Contexts
*   `context.Background()`: The root of all contexts.
*   `context.TODO()`: Placeholder when unsure which context to use.
*   `context.WithCancel(parent)`: Returns a copy that closes its Done channel when `cancel()` is called.
*   `context.WithTimeout(parent, duration)`: Cancels automatically after duration.
*   `context.WithDeadline(parent, time)`: Cancels at specific time.

## Usage Pattern
Pass `ctx` as the first argument to functions.

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

## Code Examples

**Timeout Context**

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


---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*