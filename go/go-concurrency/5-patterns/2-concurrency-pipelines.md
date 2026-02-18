---
source_course: "go-concurrency"
source_lesson: "go-concurrency-pipelines"
---

# Pipelines

A pipeline is a series of stages connected by channels, where each stage is a group of goroutines running the same function.

## Structure

```
Input -> [Stage 1] -> [Stage 2] -> [Stage 3] -> Output
```

## Example

```go
func gen(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for _, n := range nums {
            out <- n
        }
    }()
    return out
}

func square(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            out <- n * n
        }
    }()
    return out
}

// Usage
for n := range square(gen(1, 2, 3, 4)) {
    fmt.Println(n)  // 1, 4, 9, 16
}
```

## Key Rules

*   Each stage closes its output channel when done.
*   Downstream stages read until the channel is closed.

## Code Examples

**Pipeline with Filter**

```go
func filter(in <-chan int, predicate func(int) bool) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            if predicate(n) {
                out <- n
            }
        }
    }()
    return out
}

// Pipeline: generate -> filter evens -> square
nums := gen(1, 2, 3, 4, 5, 6)
evens := filter(nums, func(n int) bool { return n%2 == 0 })
result := square(evens)
```


---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*