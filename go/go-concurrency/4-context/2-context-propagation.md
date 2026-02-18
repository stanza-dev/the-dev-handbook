---
source_course: "go-concurrency"
source_lesson: "go-concurrency-context-propagation"
---

# Passing Context

## The Convention

*   Context is the first parameter, named `ctx`.
*   Don't store context in structs.
*   Don't pass nil context; use `context.TODO()` if unsure.

```go
func DoWork(ctx context.Context, args ...any) error {
    // ...
}
```

## Context Hierarchy

```go
parent := context.Background()
ctx1, cancel1 := context.WithCancel(parent)
ctx2, cancel2 := context.WithTimeout(ctx1, 10*time.Second)

// Cancelling ctx1 also cancels ctx2
cancel1()
```

## Checking Cancellation

```go
func longOperation(ctx context.Context) error {
    for i := 0; i < 100; i++ {
        select {
        case <-ctx.Done():
            return ctx.Err()
        default:
        }
        
        // Do work iteration
        doWork(i)
    }
    return nil
}
```

## Code Examples

**Retry with Context**

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


---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*