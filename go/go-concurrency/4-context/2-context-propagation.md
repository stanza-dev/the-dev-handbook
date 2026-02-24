---
source_course: "go-concurrency"
source_lesson: "go-concurrency-context-propagation"
---

# Context Propagation

## Introduction
Context is only useful if it flows through your entire call chain. Understanding how to propagate context correctly — and check for cancellation in long-running operations — is essential for building responsive, resource-efficient services.

## Key Concepts
- **Context Propagation:** Passing context as the first parameter through every function in a call chain.
- **Cancellation Checking:** Periodically checking `ctx.Done()` in long-running loops to respond to cancellation promptly.
- **Context Hierarchy:** Child contexts inherit the parent's deadline and cancellation, forming a tree.

## Real World Context
In a microservice architecture, an incoming HTTP request creates a context with a timeout. That context is passed to the database layer, the cache layer, and any outgoing HTTP calls to other services. If the original request times out, all downstream operations are cancelled in one sweep — no orphaned queries, no wasted compute.

## Deep Dive

### The Convention
- Context is the first parameter, named `ctx`.
- Do not store context in structs.
- Do not pass nil context; use `context.TODO()` if unsure.

```go
func DoWork(ctx context.Context, args ...any) error {
    // ...
}
```

### Checking Cancellation in Loops

```go
func longOperation(ctx context.Context) error {
    for i := 0; i < 100; i++ {
        select {
        case <-ctx.Done():
            return ctx.Err()
        default:
        }
        doWork(i)
    }
    return nil
}
```

The `select` with `default` is a non-blocking check. If the context is not cancelled, the default case fires immediately and the loop continues.

## Common Pitfalls
1. **Not checking ctx.Done() in long loops** — A function that runs for minutes without checking cancellation wastes resources even after the caller has given up.
2. **Passing `context.Background()` instead of the request context** — This breaks the cancellation chain. Always thread the incoming context through.

## Best Practices
1. **Check cancellation at natural breakpoints** — In loops, before expensive operations, and before I/O calls.
2. **Never pass nil for context** — Use `context.TODO()` as a placeholder if you have not yet decided which context to use.

## Summary
- Pass context as the first parameter through every function call.
- Check `ctx.Done()` in long-running loops and before expensive operations.
- Never store context in structs or pass nil.
- Use `context.TODO()` as a placeholder, not `nil`.

## Code Examples

**Retry with context cancellation — each attempt checks ctx.Done() first, aborting immediately if the context is cancelled**

```go
func fetchWithRetry(ctx context.Context, url string) (*Response, error) {
    for attempt := 0; attempt < 3; attempt++ {
        select {
        case <-ctx.Done():
            return nil, ctx.Err()
        default:
        }
        
        resp, err := fetch(ctx, url)
        if err == nil {
            return resp, nil
        }
        
        time.Sleep(time.Second * time.Duration(attempt+1))
    }
    return nil, errors.New("max retries exceeded")
}
```


## Resources

- [Go Blog: Pipelines and Cancellation](https://go.dev/blog/pipelines) — Blog post on using context for pipeline cancellation patterns

---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*