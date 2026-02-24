---
source_course: "go-concurrency"
source_lesson: "go-concurrency-scheduler-internals"
---

# The Go Scheduler (GMP Model)

## Introduction
The Go runtime includes its own scheduler that maps goroutines to OS threads. Understanding the GMP model helps you reason about performance, diagnose scheduling issues, and make informed decisions about `GOMAXPROCS`.

## Key Concepts
- **G (Goroutine):** A goroutine with its stack, instruction pointer, and state.
- **M (Machine):** An OS thread that executes goroutines.
- **P (Processor):** A logical processor holding a local run queue. The number of Ps equals `GOMAXPROCS`.
- **Work Stealing:** When a P's queue is empty, it steals goroutines from other Ps to balance load.

## Real World Context
If you deploy a Go service on a 4-core machine, the default `GOMAXPROCS` of 4 creates 4 Ps. In a container with a CPU limit of 2 cores, `GOMAXPROCS` still defaults to the host CPU count unless you use the `automaxprocs` library or set it manually, potentially wasting CPU cycles spinning up threads that get throttled.

## Deep Dive
The Go scheduler uses three entities:

1. Each **P** has a local run queue of goroutines ready to execute.
2. An **M** attaches to a **P** to execute goroutines from its queue.
3. When a goroutine blocks (I/O, channel, syscall), the M detaches from its P and parks. Another M takes over the P to keep running other goroutines.
4. If a P's local queue is empty, it **steals** half the goroutines from another P's queue.

`GOMAXPROCS` controls the number of Ps (the parallelism level):

```go
runtime.GOMAXPROCS(4)       // Use 4 logical processors
n := runtime.GOMAXPROCS(0)  // Query current value without changing it
```

The default value equals the number of CPU cores detected by the runtime.

## Common Pitfalls
1. **Setting GOMAXPROCS too high in containers** — In Docker/Kubernetes, the Go runtime sees the host CPU count, not the container limit. Use `go.uber.org/automaxprocs` to respect cgroup limits.
2. **Assuming more Ps always means faster** — For I/O-bound workloads, increasing GOMAXPROCS beyond CPU count provides no benefit and adds overhead.

## Best Practices
1. **Leave GOMAXPROCS at default for most applications** — The runtime chooses a sensible default. Only override it when profiling shows a benefit.
2. **Use `runtime.NumGoroutine()` for monitoring** — Track active goroutine count in metrics to detect leaks early.

## Summary
- G, M, P: goroutine, OS thread, logical processor with a run queue.
- GOMAXPROCS sets the number of Ps (defaults to CPU count).
- Work stealing keeps all Ps busy by redistributing goroutines.
- In containers, set GOMAXPROCS explicitly or use `automaxprocs`.

## Code Examples

**Querying runtime scheduling information — NumCPU returns hardware cores, GOMAXPROCS(0) queries without changing**

```go
import "runtime"

func main() {
    fmt.Println("CPUs:", runtime.NumCPU())
    fmt.Println("GOMAXPROCS:", runtime.GOMAXPROCS(0))
    fmt.Println("Goroutines:", runtime.NumGoroutine())
}
```


## Resources

- [Go Scheduler Design Document](https://go.dev/src/runtime/HACKING.md) — Internal documentation on how the Go runtime scheduler works

---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*