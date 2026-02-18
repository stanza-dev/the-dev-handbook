---
source_course: "go-concurrency"
source_lesson: "go-concurrency-buffered-vs-unbuffered"
---

# Channels

"Do not communicate by sharing memory; share memory by communicating."

## Unbuffered Channels
```go
ch := make(chan int)
```
A send `ch <- v` blocks until a receiver is ready `<-ch`. This provides **synchronization**.

## Buffered Channels
```go
ch := make(chan int, 3)
```
A send only blocks if the buffer is full. A receive only blocks if the buffer is empty.

## Channel Operations

```go
ch <- v     // Send v to channel
v := <-ch   // Receive from channel
v, ok := <-ch  // Receive with closed check
close(ch)   // Close channel
```

## Rules

*   Only the sender should close a channel.
*   Sending on a closed channel panics.
*   Receiving from a closed channel returns zero value and `false`.

## Code Examples

**Synchronization with Channel**

```go
func worker(done chan bool) {
    fmt.Println("Working...")
    time.Sleep(time.Second)
    fmt.Println("Done")
    done <- true
}

func main() {
    done := make(chan bool, 1)
    go worker(done)
    <-done // Block until worker sends
}
```


---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*