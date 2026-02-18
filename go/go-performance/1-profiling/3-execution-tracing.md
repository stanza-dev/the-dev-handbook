---
source_course: "go-performance"
source_lesson: "go-performance-execution-tracing"
---

# Go Tool Trace

While pprof gives statistics, the **Trace** tool gives a timeline. It visualizes goroutines, processor usage, GC pauses, and network events over time.

## Collecting a Trace

```go
import "runtime/trace"

f, _ := os.Create("trace.out")
trace.Start(f)
defer trace.Stop()
```

Or via HTTP:
```bash
curl -o trace.out http://localhost:6060/debug/pprof/trace?seconds=5
```

## Viewing

```bash
go tool trace trace.out
```

Opens a web interface with:
*   Goroutine timeline.
*   Processor utilization.
*   GC events.
*   Network/syscall blocking.

## When to Use

*   Debugging latency spikes.
*   Understanding scheduler behavior.
*   Finding GC pauses.
*   Visualizing concurrent execution.

## Code Examples

**Importing Trace**

```go
import "runtime/trace"

func main() {
    f, _ := os.Create("trace.out")
    defer f.Close()
    
    trace.Start(f)
    defer trace.Stop()
    
    // Your application code
}
```


---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*