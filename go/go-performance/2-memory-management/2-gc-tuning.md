---
source_course: "go-performance"
source_lesson: "go-performance-gc-tuning"
---

# Garbage Collector Tuning

## Introduction
Go's garbage collector runs concurrently with your application. While it requires minimal tuning for most workloads, high-throughput services benefit from understanding GOGC, GOMEMLIMIT, and the new Green Tea GC algorithm that became the default in Go 1.26.

## Key Concepts
- **GOGC**: Controls GC frequency. Default is 100 (GC triggers when heap grows 100% since last GC). Higher values = less frequent GC, more memory. `GOGC=off` disables GC.
- **GOMEMLIMIT**: A soft memory limit (e.g., `GOMEMLIMIT=4GiB`). The GC runs more aggressively as the program approaches this limit, preventing OOM kills.
- **Green Tea GC** (Go 1.26 default): A new GC algorithm that reduces garbage collection overhead by 10-40% in real-world programs. It improves marking and scanning of small objects through better locality.
- **GC pauses**: Brief stop-the-world pauses at the start and end of each GC cycle. Typically < 1ms in Go 1.26.

## Real World Context
A batch processing service with 32 GB of RAM running at GOGC=100 wastes half its time in GC. Setting `GOMEMLIMIT=28GiB` with `GOGC=off` lets the heap grow freely until it approaches the limit, then GC kicks in — cutting GC CPU overhead from 25% to under 5%.

## Deep Dive

### GOGC: Frequency Control

```bash
# Default: GC when heap doubles
GOGC=100 ./myapp

# Less frequent GC, higher memory usage
GOGC=200 ./myapp

# Disable automatic GC (must use GOMEMLIMIT or manual runtime.GC())
GOGC=off ./myapp
```

GOGC is a ratio: if the heap was 100 MB after the last GC, and GOGC=100, the next GC triggers at 200 MB.

### GOMEMLIMIT: Memory-Aware GC

Introduced in Go 1.19, GOMEMLIMIT adds a memory ceiling:

```bash
GOMEMLIMIT=4GiB ./myapp
```

When the heap approaches the limit, the GC runs more frequently regardless of GOGC. This prevents OOM kills in containerized environments where you know exactly how much memory is available.

The recommended pattern for containers:

```bash
# Container has 4 GB — reserve 500 MB for non-heap (stacks, goroutines, OS)
GOMEMLIMIT=3500MiB GOGC=100 ./myapp
```

### Green Tea GC (Go 1.26)

Go 1.26 makes the Green Tea GC the default (was experimental in Go 1.25). Key improvements:
- **10-40% reduction in GC overhead**: Improved marking and scanning of small objects through better locality and CPU scalability.
- **Vectorized scanning on modern CPUs**: ~10% additional improvement on newer amd64 CPUs (Intel Ice Lake+, AMD Zen 4+) via vector instructions.
- **Same API**: No code changes needed — you get the improvements just by upgrading to Go 1.26.
- **Opt-out**: Set `GOEXPERIMENT=nogreenteagc` if needed (expected to be removed in Go 1.27).

### Programmatic Control

```go
import "runtime/debug"

// Set GOGC programmatically
debug.SetGCPercent(200)

// Set memory limit programmatically
debug.SetMemoryLimit(4 << 30) // 4 GiB
```

## Common Pitfalls
1. **Setting GOGC too low** — GOGC=10 runs GC constantly, wasting 30-50% of CPU on garbage collection.
2. **GOGC=off without GOMEMLIMIT** — The heap grows unbounded until the OS kills the process. Always pair GOGC=off with GOMEMLIMIT.
3. **Setting GOMEMLIMIT too close to container limit** — Leave headroom for goroutine stacks and OS overhead. Use 80-90% of container memory.

## Best Practices
1. **Use GOMEMLIMIT in containers** — Set it to 80-90% of the container's memory limit. This gives the GC a clear target.
2. **Benchmark before tuning** — Profile GC overhead with `GODEBUG=gctrace=1` before changing GOGC. Many apps need no tuning.

## Summary
- GOGC controls how often GC runs (default 100 = heap doubles between cycles).
- GOMEMLIMIT caps memory usage, preventing OOM in containers.
- Go 1.26's Green Tea GC reduces GC overhead by 10-40% with no code changes.
- Pair `GOGC=off` with `GOMEMLIMIT` for maximum throughput in memory-constrained environments.
- Always measure GC overhead before tuning — use `GODEBUG=gctrace=1`.

## Code Examples

**Two GC tuning approaches — reducing frequency with GOGC or using GOMEMLIMIT for container-aware memory management**

```go
package main

import "runtime/debug"

func tuneGC() {
	// Approach 1: Reduce GC frequency for throughput
	debug.SetGCPercent(200) // GC when heap triples

	// Approach 2: Memory-bounded (ideal for containers)
	debug.SetGCPercent(-1)           // Disable GOGC-based triggering
	debug.SetMemoryLimit(3500 << 20) // 3.5 GiB soft limit

	// The GC now runs only when approaching the memory limit,
	// maximizing throughput while preventing OOM.
}
```


## Resources

- [A Guide to the Go Garbage Collector](https://go.dev/doc/gc-guide) — Official guide covering GOGC, GOMEMLIMIT, and GC internals

---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*