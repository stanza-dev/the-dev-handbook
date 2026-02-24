---
source_course: "go-concurrency"
source_lesson: "go-concurrency-errgroup-pattern"
---

# The Errgroup Pattern

## Introduction
The standard `sync.WaitGroup` does not handle errors or context cancellation. The `errgroup` package from `golang.org/x/sync` is the robust alternative for running a group of subtasks that may fail.

## Key Concepts
- **errgroup.Group:** Like WaitGroup but with error propagation and optional context cancellation.
- **First Error Wins:** `g.Wait()` returns the first non-nil error from any subtask.
- **SetLimit:** Caps the number of concurrently running goroutines (available since `errgroup` was introduced in the `x/sync` module).

## Real World Context
Fetching data from 5 external APIs to build a dashboard page is a perfect errgroup use case. All fetches run concurrently, but if any one fails, the shared context is cancelled, aborting the remaining fetches immediately instead of waiting for them to time out.

## Deep Dive
`errgroup` provides three key features:

1. **Shared Context:** When created with `WithContext`, cancels all tasks if one returns an error.
2. **Error Propagation:** Returns the first non-nil error.
3. **Concurrency Limit:** `SetLimit` controls max parallel tasks.

```go
g, ctx := errgroup.WithContext(context.Background())

g.Go(func() error {
    return fetchData(ctx)
})
g.Go(func() error {
    return processData(ctx)
})

if err := g.Wait(); err != nil {
    fmt.Println("One task failed:", err)
}
```

### With Limit

```go
g := new(errgroup.Group)
g.SetLimit(3)  // Max 3 concurrent goroutines

for _, url := range urls {
    url := url
    g.Go(func() error {
        return fetch(url)
    })
}

if err := g.Wait(); err != nil {
    log.Fatal(err)
}
```

`SetLimit` causes `g.Go` to block when the limit is reached, effectively creating a worker pool.

## Common Pitfalls
1. **Not using WithContext when you want cancellation** — `new(errgroup.Group)` does not create a shared context. Errors still propagate, but other goroutines are not cancelled.
2. **Ignoring the derived context** — When using `WithContext`, you must pass the derived `ctx` to your subtask functions, not the original parent context.

## Best Practices
1. **Use `errgroup.WithContext` for fail-fast behavior** — When one task failing means the others are pointless, this pattern cancels remaining work immediately.
2. **Use `SetLimit` instead of manual worker pools** — It is simpler and less error-prone than managing channels and WaitGroups manually.

## Summary
- `errgroup` replaces WaitGroup for error-aware concurrent tasks.
- `g.Wait()` returns the first non-nil error.
- `WithContext` creates a shared context that is cancelled on first error.
- `SetLimit` provides bounded concurrency (worker pool semantics).

## Code Examples

**Errgroup with concurrency limit — SetLimit(5) caps concurrent goroutines, g.Wait() returns the first error**

```go
import "golang.org/x/sync/errgroup"

func fetchAll(urls []string) error {
    g := new(errgroup.Group)
    g.SetLimit(5)  // Max 5 concurrent fetches
    
    for _, url := range urls {
        url := url // Capture loop var
        g.Go(func() error {
            return fetch(url)
        })
    }
    return g.Wait() // Returns first error or nil
}
```


## Resources

- [errgroup package — golang.org/x/sync](https://pkg.go.dev/golang.org/x/sync/errgroup) — Official API reference for errgroup including SetLimit and WithContext

---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*