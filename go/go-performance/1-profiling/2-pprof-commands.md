---
source_course: "go-performance"
source_lesson: "go-performance-pprof-commands"
---

# pprof Commands & Visualization

## Introduction
Collecting a profile is only half the job. You need to know how to navigate and interpret the data. The `go tool pprof` CLI offers powerful commands for drilling into hotspots, and Go 1.26 makes flame graphs the default web visualization.

## Key Concepts
- **Flat time**: Time spent directly in a function (excluding callees).
- **Cumulative time**: Time spent in a function *including* all its callees.
- **Flame graph**: A visualization where wider bars mean more samples. In Go 1.26, `go tool pprof -http` defaults to the flame graph view.
- **Call graph**: A directed graph showing function call relationships weighted by sample counts.

## Real World Context
When investigating a latency spike, you might collect a CPU profile and use `top` to find the most expensive function, then `list` to see which lines cost the most. For memory leaks, `top -cum` on a heap profile reveals which call chains allocate the most.

## Deep Dive

### Interactive CLI Commands

After opening a profile with `go tool pprof cpu.prof`, use these commands:

```
(pprof) top 10          # Top 10 functions by flat time
(pprof) top -cum 10     # Top 10 by cumulative time
(pprof) list funcName   # Source-level annotation for a function
(pprof) peek funcName   # Show callers and callees
(pprof) web             # Open call graph in browser
```

The `top` command shows where time is actually spent. Use `top -cum` when you suspect a high-level function is slow because of something it calls.

### Web-Based Visualization

Launch the interactive web UI:

```bash
go tool pprof -http=:8080 cpu.prof
```

In Go 1.26, this defaults to a **flame graph** view — the most intuitive way to understand where time is spent. You can click on any bar to zoom into that subtree.

### Comparing Profiles

To validate an optimization, compare before and after:

```bash
go tool pprof -diff_base=before.prof after.prof
```

Red sections grew (got worse), green sections shrank (improved). This is invaluable for regression testing.

## Common Pitfalls
1. **Looking only at flat time** — A function with low flat time but high cumulative time might be the real bottleneck because of what it calls.
2. **Ignoring inlined functions** — The Go compiler inlines small functions. They may not appear in profiles. Use `go build -gcflags='-m'` to check what gets inlined.

## Best Practices
1. **Start with flame graphs** — They give the best overview. Drill into specific functions with `list` after identifying hot paths.
2. **Profile realistic workloads** — Synthetic benchmarks may not reflect production behavior. Use production profiles when possible.

## Summary
- `top` and `top -cum` show the most expensive functions by flat and cumulative time.
- `list` provides source-level line-by-line cost annotations.
- Go 1.26 defaults to flame graph visualization in the web UI.
- Use `-diff_base` to compare profiles and validate optimizations.

## Code Examples

**Common pprof workflows — collecting profiles, launching the web UI, and comparing before/after**

```bash
# Collect a 30-second CPU profile from a running server
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30

# Open interactive web UI with flame graph (default in Go 1.26)
go tool pprof -http=:8080 cpu.prof

# Compare two profiles to validate an optimization
go tool pprof -diff_base=before.prof after.prof
```


## Resources

- [Diagnostics - The Go Programming Language](https://go.dev/doc/diagnostics) — Official Go documentation on diagnostics including pprof usage

---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*