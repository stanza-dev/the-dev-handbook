---
source_course: "go-concurrency"
source_lesson: "go-concurrency-goroutine-leaks"
---

# Goroutine Leaks

A goroutine leaks when it is started but never terminates. This consumes memory and can crash the program eventually.

## Common Causes
1.  **Blocked Send:** Sending to a channel that has no receiver.
2.  **Blocked Receive:** Waiting on a channel that will never send or close.
3.  **Nil Channel:** Sending or receiving from a nil channel blocks forever.
4.  **Infinite Loop:** A loop without exit condition or proper cancellation.

## Detection

```go
fmt.Println("Active goroutines:", runtime.NumGoroutine())
```

## Solutions

*   Always ensure there is a path for the goroutine to exit.
*   Use `context` for cancellation.
*   Close channels when done sending.
*   Use buffered channels when appropriate.

## Code Examples

**Leaking Goroutine**

```go
func leak() {
    ch := make(chan int)
    go func() {
        val := <-ch // Blocks forever because nothing sends to ch
        fmt.Println(val)
    }()
}
// The anonymous goroutine is leaked.
```


---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*