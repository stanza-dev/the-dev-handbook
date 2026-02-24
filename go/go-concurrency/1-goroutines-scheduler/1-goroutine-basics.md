---
source_course: "go-concurrency"
source_lesson: "go-concurrency-goroutine-basics"
---

# Goroutines vs Threads

## Introduction
Go achieves massive concurrency not by mapping one goroutine to one OS thread, but by multiplexing many goroutines onto a small set of threads. Understanding this distinction is fundamental to writing efficient concurrent Go code.

## Key Concepts
- **Goroutine:** A lightweight unit of execution managed by the Go runtime, starting with a ~2 KB stack that grows dynamically.
- **OS Thread:** A kernel-level thread with a fixed ~1 MB stack, expensive to create and context-switch.
- **M:N Scheduling:** Go maps M goroutines onto N OS threads, letting thousands of goroutines share a handful of threads.

## Real World Context
In a production HTTP server, every incoming request typically spawns a goroutine. Because goroutines are so cheap, Go servers routinely handle tens of thousands of concurrent connections without the memory overhead that would cripple a thread-per-request model in languages like Java or C++.

## Deep Dive
Go does not use one OS thread per goroutine. Instead, it uses an M:N scheduler (M goroutines on N OS threads).

- **Lightweight:** Goroutines start with a 2 KB stack (vs. 1 MB+ for threads).
- **Dynamic:** Stacks grow and shrink as needed via contiguous stack copying.
- **Cooperative:** The runtime switches contexts during blocking operations (I/O, channel sends/receives, sleep) or tight loops (via async preemption since Go 1.14).

Starting a goroutine is as simple as placing `go` before a function call:

```go
go doWork()
go func() {
    // Anonymous goroutine
}()
```

**Warning:** The `main` function is itself a goroutine. When it exits, the program terminates, killing all other goroutines immediately without cleanup.

## Common Pitfalls
1. **Forgetting main exits immediately** — If main returns before background goroutines finish, their work is lost. Always use `sync.WaitGroup` or channels to synchronize.
2. **Assuming goroutines run in order** — The scheduler is non-deterministic. Never rely on goroutine execution order without explicit synchronization.

## Best Practices
1. **Always have a completion signal** — Use a `WaitGroup`, channel, or `context` so main knows when goroutines finish.
2. **Keep goroutines short-lived** — Long-running goroutines increase the risk of leaks. Prefer goroutines that do one job and exit.

## Summary
- Goroutines are lightweight (~2 KB stack) compared to OS threads (~1 MB).
- Go uses M:N scheduling to multiplex goroutines onto threads.
- The `go` keyword launches a goroutine, but main must wait for them to finish.
- Async preemption (since Go 1.14) prevents tight loops from starving the scheduler.

## Code Examples

**Launching a goroutine — without synchronization, main may exit before the goroutine prints**

```go
func main() {
    go fmt.Println("In background")
    fmt.Println("In main")
    // Without a sleep or sync, main might exit before background runs
    time.Sleep(100 * time.Millisecond)
}
```


## Resources

- [The Go Blog: Concurrency is not Parallelism](https://go.dev/blog/waza-talk) — Rob Pike's foundational talk on the difference between concurrency and parallelism in Go
- [Effective Go: Goroutines](https://go.dev/doc/effective_go#goroutines) — Official guide on goroutine basics and design philosophy

---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*