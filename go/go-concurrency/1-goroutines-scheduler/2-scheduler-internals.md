---
source_course: "go-concurrency"
source_lesson: "go-concurrency-scheduler-internals"
---

# GMP Model

The Go scheduler uses three entities:

*   **G (Goroutine):** A goroutine with its stack and instruction pointer.
*   **M (Machine):** An OS thread that executes goroutines.
*   **P (Processor):** A logical processor with a run queue of goroutines.

## How It Works

1.  Each P has a local run queue of goroutines.
2.  M attaches to a P to execute goroutines.
3.  When a goroutine blocks (I/O, channel), M detaches from P and waits.
4.  Another M takes over the P to continue running other goroutines.

## GOMAXPROCS

Controls the number of Ps (parallelism level):

```go
runtime.GOMAXPROCS(4)  // Use 4 logical processors
n := runtime.GOMAXPROCS(0)  // Query current value
```

Default: Number of CPU cores.

## Work Stealing

If a P's queue is empty, it steals work from other Ps to balance load.

## Code Examples

**Runtime Information**

```go
import "runtime"

func main() {
    fmt.Println("CPUs:", runtime.NumCPU())
    fmt.Println("GOMAXPROCS:", runtime.GOMAXPROCS(0))
    fmt.Println("Goroutines:", runtime.NumGoroutine())
}
```


---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*