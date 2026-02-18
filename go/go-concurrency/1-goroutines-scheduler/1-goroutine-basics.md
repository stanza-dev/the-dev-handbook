---
source_course: "go-concurrency"
source_lesson: "go-concurrency-goroutine-basics"
---

# The M:N Scheduler

Go does not use one OS thread per goroutine. Instead, it uses an M:N scheduler (M goroutines on N OS threads).

*   **Lightweight:** Goroutines start with a 2KB stack (vs. 1MB+ for threads).
*   **Dynamic:** Stacks grow and shrink as needed.
*   **Cooperative:** The runtime switches contexts during blocking operations (I/O, channel sends/receives, sleep) or tight loops (via async preemption in recent Go versions).

## Starting a Goroutine
Simply put `go` before a function call.

```go
go doWork()
go func() {
    // Anonymous goroutine
}()
```

**Warning:** The `main` function is a goroutine. When it exits, the program terminates, killing all other goroutines immediately.

## Code Examples

**Basic Goroutine**

```go
func main() {
    go fmt.Println("In background")
    fmt.Println("In main")
    // Without a sleep or sync, main might exit before background runs
    time.Sleep(100 * time.Millisecond)
}
```


---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*