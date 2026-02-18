---
source_course: "go-performance"
source_lesson: "go-performance-runtime-metrics"
---

# Runtime Statistics

Go provides runtime metrics without profiling overhead.

## runtime.MemStats

```go
var m runtime.MemStats
runtime.ReadMemStats(&m)

fmt.Printf("Alloc = %v MiB\n", m.Alloc / 1024 / 1024)
fmt.Printf("TotalAlloc = %v MiB\n", m.TotalAlloc / 1024 / 1024)
fmt.Printf("Sys = %v MiB\n", m.Sys / 1024 / 1024)
fmt.Printf("NumGC = %v\n", m.NumGC)
```

## Key Fields

*   `Alloc`: Currently allocated heap memory.
*   `TotalAlloc`: Total bytes allocated (cumulative).
*   `Sys`: Total memory obtained from OS.
*   `NumGC`: Number of GC cycles.
*   `PauseTotalNs`: Total GC pause time.

## runtime/metrics (Go 1.16+)

More efficient and comprehensive:

```go
import "runtime/metrics"

samples := []metrics.Sample{
    {Name: "/gc/heap/allocs:bytes"},
    {Name: "/gc/heap/goal:bytes"},
}
metrics.Read(samples)
```

## Code Examples

**Memory Statistics**

```go
func printMemStats() {
    var m runtime.MemStats
    runtime.ReadMemStats(&m)
    
    fmt.Printf("Heap: %d MB\n", m.HeapAlloc/1024/1024)
    fmt.Printf("Stack: %d MB\n", m.StackInuse/1024/1024)
    fmt.Printf("GC Cycles: %d\n", m.NumGC)
}
```


---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*