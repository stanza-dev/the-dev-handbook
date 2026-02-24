---
source_course: "go-performance"
source_lesson: "go-performance-pprof-tool"
---

# Profiling with pprof

## Introduction
Performance optimization begins with measurement. Go ships with `pprof`, a built-in profiling tool that lets you pinpoint exactly where your program spends CPU time and allocates memory. Without profiling, you are guessing — and guesses are almost always wrong.

## Key Concepts
- **Profile**: A statistical sample of program behavior over time (CPU usage, memory allocations, blocking events).
- **pprof**: Go's built-in profiling tool, accessed via `runtime/pprof` (for CLI tools) or `net/http/pprof` (for long-running servers).
- **Sample**: A single snapshot of the call stack, collected at regular intervals (default: 100 Hz for CPU profiles).
- **Flame Graph**: A visualization where the x-axis shows cumulative time and the y-axis shows call depth. In Go 1.26, flame graphs are the default pprof view.

## Real World Context
Every production Go service should expose pprof endpoints. When a service starts consuming unexpected CPU or memory, pprof profiles are the first tool an SRE reaches for. Companies like Google, Uber, and Cloudflare use pprof daily to diagnose production performance issues.

## Deep Dive

### Enabling pprof for HTTP Servers

The simplest way to add profiling to a server is a blank import:

```go
import _ "net/http/pprof"

func main() {
    // Register pprof handlers on DefaultServeMux
    go http.ListenAndServe(":6060", nil)
    // ... your application code
}
```

This registers handlers under `/debug/pprof/`. You can then collect a 30-second CPU profile:

```bash
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30
```

### Profile Types

Go provides several profile types, each targeting a different bottleneck:

- `/debug/pprof/profile` — CPU: where the program spends compute time.
- `/debug/pprof/heap` — Memory: which functions allocate and how much is retained.
- `/debug/pprof/goroutine` — Goroutines: all goroutine stack traces (use for leak detection).
- `/debug/pprof/block` — Blocking: where goroutines block on synchronization primitives.
- `/debug/pprof/mutex` — Mutex contention: which mutexes are most contended.

### Programmatic Profiling for CLI Tools

For non-server programs, use `runtime/pprof` directly:

```go
f, _ := os.Create("cpu.prof")
pprof.StartCPUProfile(f)
defer pprof.StopCPUProfile()
```

This writes a profile file that you can analyze offline.

## Common Pitfalls
1. **Profiling in production without rate limiting** — CPU profiling adds ~5% overhead. Use short durations (10-30s) and avoid running multiple profiles simultaneously.
2. **Forgetting to enable the block/mutex profiles** — These are disabled by default. Set `runtime.SetBlockProfileRate(1)` and `runtime.SetMutexProfileFraction(1)` before collecting.
3. **Exposing pprof on a public port** — Always bind pprof to a separate, internal-only port. It can reveal sensitive information about your application.

## Best Practices
1. **Always profile before optimizing** — Measure first, then optimize the hottest paths. A 10x improvement on a function that uses 1% of CPU time is negligible.
2. **Compare before/after profiles** — Use `go tool pprof -diff_base=before.prof after.prof` to verify your optimization actually helped.

## Summary
- Go includes `pprof` for CPU, memory, goroutine, block, and mutex profiling.
- For servers, use `net/http/pprof`; for CLI tools, use `runtime/pprof`.
- Always measure before optimizing — intuition about bottlenecks is unreliable.
- In Go 1.26, flame graphs are now the default visualization in `go tool pprof -http`.
- Keep pprof endpoints internal and profile for short durations in production.

## Code Examples

**Enabling pprof on a separate internal port — the blank import registers all profiling handlers on DefaultServeMux**

```go
package main

import (
	"log"
	"net/http"
	_ "net/http/pprof" // Registers /debug/pprof/ handlers
)

func main() {
	// Expose pprof on a separate internal port
	go func() {
		log.Println(http.ListenAndServe("localhost:6060", nil))
	}()

	// Your application server on the public port
	mux := http.NewServeMux()
	mux.HandleFunc("/", handleRequest)
	http.ListenAndServe(":8080", mux)
}
```


## Resources

- [Profiling Go Programs](https://go.dev/blog/pprof) — Official Go blog post on using pprof for CPU and memory profiling
- [runtime/pprof Package](https://pkg.go.dev/runtime/pprof) — API reference for programmatic profiling in Go

---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*