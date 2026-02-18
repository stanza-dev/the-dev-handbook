---
source_course: "go-performance"
source_lesson: "go-performance-gc-tuning"
---

# Go's Garbage Collector

Go uses a concurrent, tri-color mark-and-sweep GC with very low pause times (usually <1ms).

## GOGC

Controls GC frequency. Default is 100, meaning GC runs when heap grows 100% since last GC.

```bash
GOGC=50   # Run more often (less memory, more CPU)
GOGC=200  # Run less often (more memory, less CPU)
GOGC=off  # Disable GC (dangerous!)
```

## GOMEMLIMIT (Go 1.19+)

Sets a soft memory limit:

```bash
GOMEMLIMIT=1GiB  # Try to stay under 1GB
```

## Manual GC Control

```go
runtime.GC()  // Force GC cycle
debug.FreeOSMemory()  // Return memory to OS
```

## GC Pauses

Monitor with:
```go
debug.SetGCPercent(100)  // Same as GOGC
stats := debug.GCStats{}
debug.ReadGCStats(&stats)
fmt.Println(stats.PauseTotal)
```

## Code Examples

**Memory Limit**

```bash
# Set memory limit to 512MB
GOMEMLIMIT=512MiB ./myapp

# Or in code
debug.SetMemoryLimit(512 << 20)
```


---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*