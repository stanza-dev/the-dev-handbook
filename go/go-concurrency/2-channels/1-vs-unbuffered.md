---
source_course: "go-concurrency"
source_lesson: "go-concurrency-buffered-vs-unbuffered"
---

# Channel Semantics

## Introduction
Channels are Go's primary mechanism for communication between goroutines. The famous Go proverb says: "Do not communicate by sharing memory; share memory by communicating." Understanding channel semantics — buffered vs. unbuffered, blocking behavior, and closing — is essential.

## Key Concepts
- **Unbuffered Channel:** A channel with no buffer. A send blocks until a receiver is ready, providing synchronization.
- **Buffered Channel:** A channel with a fixed-size buffer. A send blocks only when the buffer is full; a receive blocks only when the buffer is empty.
- **Channel Close:** Signals that no more values will be sent. Receiving from a closed channel returns the zero value immediately.

## Real World Context
In a web crawler, an unbuffered channel between the URL dispatcher and worker goroutines ensures natural backpressure: if all workers are busy, the dispatcher blocks until one is free. A buffered channel would allow the dispatcher to queue URLs ahead of time, trading memory for throughput.

## Deep Dive

### Unbuffered Channels
```go
ch := make(chan int)
```
A send `ch <- v` blocks until a receiver is ready via `<-ch`. This provides a synchronization point — the sender knows the receiver has the value.

### Buffered Channels
```go
ch := make(chan int, 3)
```
A send only blocks if the buffer is full. A receive only blocks if the buffer is empty.

### Channel Operations

```go
ch <- v         // Send v to channel
v := <-ch       // Receive from channel
v, ok := <-ch   // Receive with closed check
close(ch)       // Close channel
```

### Rules
- Only the sender should close a channel.
- Sending on a closed channel panics.
- Receiving from a closed channel returns the zero value and `false` for the ok flag.

## Common Pitfalls
1. **Closing a channel from the receiver side** — Only the sender knows when there are no more values to send. Closing from the receiver risks a send-on-closed-channel panic.
2. **Using a buffered channel to avoid deadlocks** — A buffer only delays the deadlock if the root cause is a missing receiver. Fix the design, not the buffer size.
3. **Nil channels block forever** — Sending to or receiving from a `nil` channel blocks forever. This is useful in `select` statements (setting a channel to `nil` disables that case), but dangerous if accidental. A `var ch chan int` declaration without `make` leaves the channel nil.
4. **Double-close panics** — Closing an already-closed channel causes a runtime panic. Similarly, closing a `nil` channel also panics. Always ensure a channel is closed exactly once, typically by the sender using patterns like `sync.Once` or a dedicated close goroutine.

## Best Practices
1. **Default to unbuffered channels** — They make synchronization behavior explicit. Only add a buffer when profiling shows a throughput benefit.
2. **Use the comma-ok idiom** — Always check `v, ok := <-ch` when the channel might be closed to distinguish zero values from closed-channel reads.

## Summary
- Unbuffered channels synchronize sender and receiver.
- Buffered channels decouple sender and receiver up to the buffer size.
- Only the sender should close a channel; sending on a closed channel panics.
- Use the comma-ok idiom to detect channel closure.

## Code Examples

**Using a channel for synchronization — main blocks until the worker signals completion by sending to the done channel**

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


## Resources

- [Effective Go: Channels](https://go.dev/doc/effective_go#channels) — Official guide to channel usage patterns and idioms

---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*