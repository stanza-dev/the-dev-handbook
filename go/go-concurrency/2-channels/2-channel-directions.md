---
source_course: "go-concurrency"
source_lesson: "go-concurrency-channel-directions"
---

# Channel Direction Types

You can specify channel direction in function signatures for type safety.

```go
func producer(out chan<- int) {  // Send-only
    out <- 42
}

func consumer(in <-chan int) {   // Receive-only
    val := <-in
    fmt.Println(val)
}
```

## Benefits

*   Compile-time safety: Can't accidentally send to a receive-only channel.
*   Documentation: Clear intent in function signature.
*   API design: Restrict what callers can do.

## Conversion

Bidirectional channels can be converted to directional:

```go
ch := make(chan int)
var sendOnly chan<- int = ch  // OK
var recvOnly <-chan int = ch  // OK
```

But not the other way around!

## Code Examples

**Directional Channels**

```go
func produce(numbers chan<- int) {
    for i := 0; i < 5; i++ {
        numbers <- i
    }
    close(numbers)
}

func consume(numbers <-chan int) {
    for n := range numbers {
        fmt.Println(n)
    }
}
```


---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*