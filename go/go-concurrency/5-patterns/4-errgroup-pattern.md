---
source_course: "go-concurrency"
source_lesson: "go-concurrency-errgroup-pattern"
---

# golang.org/x/sync/errgroup

Standard `WaitGroup` doesn't handle errors or context cancellation. `errgroup` is the robust alternative for running a group of subtasks.

## Features
1.  **Shared Context:** Cancels all other tasks if one returns an error.
2.  **Error Propagation:** Returns the first non-nil error.
3.  **Concurrency Limit:** `SetLimit` controls max parallel tasks.

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

## With Limit (Go 1.19+)

```go
g := new(errgroup.Group)
g.SetLimit(3)  // Max 3 concurrent goroutines

for _, url := range urls {
    url := url
    g.Go(func() error {
        return fetch(url)
    })
}
```

## Code Examples

**Errgroup with Limit**

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
    return g.Wait()
}
```


---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*