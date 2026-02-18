---
source_course: "go-std-lib"
source_lesson: "go-std-lib-tickers-timers"
---

# Scheduling Events

## Timers
Fire once in the future.

```go
timer := time.NewTimer(2 * time.Second)
<-timer.C
fmt.Println("Timer fired")

// Or cancel it
if !timer.Stop() {
    <-timer.C  // Drain the channel if already fired
}
```

## time.After

Convenience for one-shot timeouts:

```go
select {
case <-time.After(1 * time.Second):
    fmt.Println("Timed out")
case result := <-ch:
    fmt.Println("Got result", result)
}
```

## Tickers
Fire repeatedly at an interval.

```go
ticker := time.NewTicker(1 * time.Second)
defer ticker.Stop() // Important to stop to release resources!

for t := range ticker.C {
    fmt.Println("Tick at", t)
}
```

## time.Tick (Convenience)

```go
// Warning: Cannot be stopped, may leak!
for t := range time.Tick(1 * time.Second) {
    fmt.Println("Tick at", t)
}
```

## Code Examples

**One-shot Timeout**

```go
select {
case <-time.After(1 * time.Second):
    fmt.Println("Timed out")
}
```


---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*