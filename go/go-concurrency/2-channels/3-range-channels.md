---
source_course: "go-concurrency"
source_lesson: "go-concurrency-range-channels"
---

# Iterating Channels

You can use `for range` to iterate over values received from a channel. The loop terminates when the channel is **closed**.

```go
ch := make(chan int, 2)
ch <- 1
ch <- 2
close(ch)

for v := range ch {
    fmt.Println(v)
}
```

## Critical: Always Close

If the channel is never closed, the loop blocks forever (deadlock if main thread, or leak if background goroutine).

## Producer Pattern

```go
func generate(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)  // Always close!
        for _, n := range nums {
            out <- n
        }
    }()
    return out
}
```

## Code Examples

**Producer Consumer**

```go
func producer(out chan<- int) {
    for i := 0; i < 5; i++ {
        out <- i
    }
    close(out) // Essential!
}

func main() {
    ch := make(chan int)
    go producer(ch)
    for n := range ch {
        fmt.Println(n)
    }
}
```


---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*