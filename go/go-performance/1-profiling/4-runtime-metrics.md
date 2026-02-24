---
source_course: "go-performance"
source_lesson: "go-performance-runtime-metrics"
---

# Runtime Metrics & Diagnostics

## Introduction
Beyond profiling, Go provides runtime metrics for continuous monitoring. These let you track GC behavior, goroutine counts, and memory usage in real time — essential for detecting performance regressions before they become outages.

## Key Concepts
- **runtime/metrics**: The standard package for reading runtime statistics (GC pauses, heap size, goroutine count).
- **GODEBUG**: An environment variable that enables runtime debugging output (GC logging, scheduler tracing).
- **expvar**: A package that exposes application metrics as JSON via HTTP.
- **Goroutine leak profile** (Go 1.26, experimental): A new pprof profile type that detects goroutines blocked on unreachable concurrency primitives. Requires `GOEXPERIMENT=goroutineleakprofile`.

## Real World Context
In production, you cannot attach a profiler every time something goes wrong. Instead, you export runtime metrics to Prometheus, Datadog, or similar systems. When the heap size doubles or goroutine count climbs steadily, you get an alert before the OOM killer strikes.

## Deep Dive

### runtime/metrics Package

The `runtime/metrics` package provides structured access to runtime statistics:

```go
import "runtime/metrics"

// Read specific metrics
samples := []metrics.Sample{
    {Name: "/gc/pauses:seconds"},
    {Name: "/memory/classes/heap/objects:bytes"},
    {Name: "/sched/goroutines:goroutines"},
}
metrics.Read(samples)
```

This is more reliable than `runtime.ReadMemStats()` because it does not stop-the-world.

### GODEBUG for Diagnostic Output

The `GODEBUG` environment variable enables diagnostic logging:

```bash
# Log every GC cycle with pause times
GODEBUG=gctrace=1 ./myapp

# Log scheduler events (very verbose)
GODEBUG=schedtrace=1000 ./myapp
```

The `gctrace=1` output shows heap size, GC pause duration, and CPU utilization per cycle — invaluable for tuning GOGC.

### Goroutine Leak Detection (Go 1.26, Experimental)

Go 1.26 introduces an experimental goroutine leak profile (requires `GOEXPERIMENT=goroutineleakprofile`) that detects goroutines blocked on unreachable concurrency primitives:

```bash
# Build with: GOEXPERIMENT=goroutineleakprofile go build
go tool pprof http://localhost:6060/debug/pprof/goroutineleak
```

It detects goroutines blocked on channels, mutexes, or sync.Cond where the primitive itself is unreachable — a strong signal of a leak. This feature is expected to be enabled by default in Go 1.27.

### expvar for Application Metrics

```go
import "expvar"

var requestCount = expvar.NewInt("requests_total")

func handler(w http.ResponseWriter, r *http.Request) {
    requestCount.Add(1)
    // ...
}
// Metrics available at /debug/vars
```

## Common Pitfalls
1. **Relying on runtime.NumGoroutine() alone for leak detection** — The number fluctuates normally. Use the goroutine profile or Go 1.26's leak profile for accurate detection.
2. **Using runtime.ReadMemStats() in hot paths** — It stops the world briefly. Use `runtime/metrics` instead for non-blocking reads.

## Best Practices
1. **Export metrics to your monitoring system** — Use `runtime/metrics` to feed Prometheus or Datadog. Set alerts on heap growth rate and goroutine count.
2. **Enable gctrace in staging** — Run with `GODEBUG=gctrace=1` during load tests to understand GC behavior under realistic load.

## Summary
- `runtime/metrics` provides non-blocking access to GC, heap, and goroutine statistics.
- `GODEBUG=gctrace=1` logs GC cycle details; `schedtrace=1000` logs scheduler state.
- Go 1.26 adds an experimental goroutine leak profile (requires `GOEXPERIMENT=goroutineleakprofile`) for detecting goroutines blocked on unreachable primitives.
- `expvar` provides simple HTTP-accessible application metrics.
- Always monitor runtime metrics in production to catch performance regressions early.

## Code Examples

**Reading runtime metrics non-blockingly — unlike runtime.ReadMemStats(), this does not stop the world**

```go
package main

import (
	"fmt"
	"runtime/metrics"
)

func printRuntimeMetrics() {
	// Define which metrics to read
	samples := []metrics.Sample{
		{Name: "/gc/pauses:seconds"},
		{Name: "/memory/classes/heap/objects:bytes"},
		{Name: "/sched/goroutines:goroutines"},
	}

	// Read without stopping the world
	metrics.Read(samples)

	for _, s := range samples {
		fmt.Printf("%s = %v\n", s.Name, s.Value)
	}
}
```


## Resources

- [runtime/metrics Package](https://pkg.go.dev/runtime/metrics) — API reference for Go's structured runtime metrics
- [Go Runtime GODEBUG Settings](https://pkg.go.dev/runtime#hdr-Environment_Variables) — Documentation on GODEBUG and other runtime environment variables

---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*