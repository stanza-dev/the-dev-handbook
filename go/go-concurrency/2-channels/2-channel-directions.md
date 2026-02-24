---
source_course: "go-concurrency"
source_lesson: "go-concurrency-channel-directions"
---

# Directional Channels

## Introduction
Go lets you restrict a channel to send-only or receive-only in function signatures. This provides compile-time safety and makes the data flow direction explicit in your API design.

## Key Concepts
- **Send-only Channel (`chan<- T`):** Can only be used to send values. Attempting to receive from it is a compile error.
- **Receive-only Channel (`<-chan T`):** Can only be used to receive values. Attempting to send to it is a compile error.
- **Bidirectional Channel (`chan T`):** Can be used for both sending and receiving. Implicitly converts to either directional type.

## Real World Context
In a pipeline architecture, each stage function accepts a receive-only input channel and returns a send-only output channel. This makes the data flow direction explicit at every boundary and prevents accidental misuse — a consumer function literally cannot close or send to its input channel.

## Deep Dive
You can specify channel direction in function signatures for type safety:

```go
func producer(out chan<- int) {  // Send-only
    out <- 42
}

func consumer(in <-chan int) {   // Receive-only
    val := <-in
    fmt.Println(val)
}
```

### Benefits
- **Compile-time safety:** You cannot accidentally send to a receive-only channel.
- **Documentation:** The function signature clearly communicates intent.
- **API design:** You restrict what callers can do with the channel.

### Conversion
Bidirectional channels can be converted to directional:

```go
ch := make(chan int)
var sendOnly chan<- int = ch  // OK
var recvOnly <-chan int = ch  // OK
```

But not the other way around — you cannot convert a directional channel back to bidirectional.

## Common Pitfalls
1. **Passing bidirectional channels everywhere** — Without directional types, any function can accidentally close or misuse a channel. Always narrow the type in function signatures.
2. **Trying to convert directional to bidirectional** — This is a compile error. Design your code so the bidirectional channel is created once and narrowed at each call site.

## Best Practices
1. **Always use directional types in function parameters** — It documents intent and catches bugs at compile time.
2. **Return receive-only channels from generator functions** — A function that produces values should return `<-chan T` so callers cannot close or send to it.

## Summary
- `chan<- T` is send-only; `<-chan T` is receive-only.
- Bidirectional channels implicitly convert to directional, but not vice versa.
- Use directional types in function signatures for compile-time safety.
- Return `<-chan T` from generator/producer functions.

## Code Examples

**Producer accepts send-only, consumer accepts receive-only — the compiler enforces correct usage at each boundary**

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


## Resources

- [Go Spec: Channel Types](https://go.dev/ref/spec#Channel_types) — Language specification for channel direction types and conversion rules

---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*