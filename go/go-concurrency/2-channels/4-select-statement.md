---
source_course: "go-concurrency"
source_lesson: "go-concurrency-select-statement"
---

# Select

`select` lets a goroutine wait on multiple channel operations.

```go
select {
case v := <-ch1:
    fmt.Println("From ch1:", v)
case v := <-ch2:
    fmt.Println("From ch2:", v)
case ch3 <- 42:
    fmt.Println("Sent to ch3")
default:
    fmt.Println("No communication ready")
}
```

## Rules

*   If multiple cases are ready, one is chosen at random.
*   `default` executes immediately if no other case is ready.
*   Without `default`, select blocks until a case is ready.

## Common Patterns

### Timeout

```go
select {
case v := <-ch:
    process(v)
case <-time.After(1 * time.Second):
    fmt.Println("Timeout")
}
```

### Non-blocking Send/Receive

```go
select {
case ch <- v:
    fmt.Println("Sent")
default:
    fmt.Println("Channel full")
}
```

## Code Examples

**Select Statement**

```go
func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)
    
    go func() {
        time.Sleep(100 * time.Millisecond)
        ch1 <- "one"
    }()
    go func() {
        time.Sleep(200 * time.Millisecond)
        ch2 <- "two"
    }()
    
    for i := 0; i < 2; i++ {
        select {
        case msg := <-ch1:
            fmt.Println(msg)
        case msg := <-ch2:
            fmt.Println(msg)
        }
    }
}
```


---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*