---
source_course: "go-performance"
source_lesson: "go-performance-execution-tracing"
---

# Execution Tracing

## Introduction
While pprof tells you *where* time is spent, execution tracing tells you *when* and *why*. The `runtime/trace` package captures a timeline of goroutine scheduling, GC pauses, network I/O, and system calls — letting you see the full story of what your program did over time.

## Key Concepts
- **Execution trace**: A timestamped log of runtime events (goroutine creation, blocking, GC, syscalls).
- **runtime/trace**: The standard library package for capturing traces programmatically.
- **Trace viewer**: A web-based timeline UI launched with `go tool trace`.
- **FlightRecorder** (Go 1.25+): A ring-buffer tracer in `runtime/trace` that continuously records the last N seconds, only saving data when an interesting event triggers it.

## Real World Context
Imagine a request that is fast on average but occasionally takes 500ms. A CPU profile shows nothing unusual because the slow path is rare. An execution trace captures the exact timeline: you might discover a goroutine was descheduled waiting for a GC pause, or was blocked on a channel for 400ms.

## Deep Dive

### Capturing a Trace

For HTTP servers, collect a 5-second trace:

```bash
curl -o trace.out http://localhost:6060/debug/pprof/trace?seconds=5
go tool trace trace.out
```

For CLI tools, capture programmatically:

```go
f, _ := os.Create("trace.out")
trace.Start(f)
defer trace.Stop()
```

### What Traces Reveal

The trace viewer shows:
- **Goroutine timeline**: When each goroutine was running, blocked, or waiting.
- **Processor (P) timeline**: What each logical processor was doing.
- **GC events**: When GC ran and how long it paused.
- **Network/syscall blocking**: When goroutines were blocked on I/O.

### FlightRecorder (Go 1.25+)

The `runtime/trace.FlightRecorder` maintains a rolling buffer of trace data. When something goes wrong, you snapshot the buffer — capturing what happened *before* the problem:

```go
fr := trace.NewFlightRecorder()
fr.Start()

// When something interesting happens...
snapshot, _ := fr.WriteTo(f)
```

This is invaluable for debugging intermittent production issues without the overhead of continuous tracing.

## Common Pitfalls
1. **Traces are large** — A 5-second trace can be hundreds of MB. Keep traces short and targeted.
2. **Trace overhead is higher than pprof** — Expect 10-25% overhead during trace collection. Never leave it running in production.
3. **Confusing traces with profiles** — Traces show *when* things happen (timeline). Profiles show *where* time is spent (aggregated). Use both.

## Best Practices
1. **Use traces for latency investigation** — When pprof shows nothing interesting but latency is high, traces reveal scheduling delays, GC pauses, and blocking.
2. **Use FlightRecorder for production** — It has minimal overhead until you snapshot, making it safe for always-on deployment.

## Summary
- Execution traces capture a timeline of goroutine scheduling, GC, and I/O events.
- Use `go tool trace` to visualize trace data as an interactive timeline.
- Traces complement profiles: profiles show *where*, traces show *when* and *why*.
- Go 1.25 added `FlightRecorder` for low-overhead, always-on tracing with snapshot support.

## Code Examples

**Programmatic trace collection — captures all runtime events during doWork() for timeline analysis**

```go
package main

import (
	"os"
	"runtime/trace"
)

func main() {
	// Create a trace output file
	f, err := os.Create("trace.out")
	if err != nil {
		panic(err)
	}
	defer f.Close()

	// Start tracing — captures goroutine scheduling, GC, I/O
	trace.Start(f)
	defer trace.Stop()

	// Your application code runs here...
	doWork()
}

// Analyze with: go tool trace trace.out
```


## Resources

- [Go Execution Tracer](https://go.dev/blog/execution-traces-2024) — Go blog post on the redesigned execution tracer and its capabilities
- [runtime/trace Package](https://pkg.go.dev/runtime/trace) — API reference for Go's execution tracing including FlightRecorder

---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*