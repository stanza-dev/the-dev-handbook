---
source_course: "go-performance"
source_lesson: "go-performance-pgo-benchmarks"
---

# Measuring PGO Impact

## Introduction
PGO is only valuable if it demonstrably improves performance. Rigorous benchmarking before and after PGO is essential — both to justify the operational overhead of profile collection and to detect cases where PGO provides minimal benefit.

## Key Concepts
- **A/B benchmarking**: Comparing the same workload between a PGO-enabled and PGO-disabled build.
- **benchstat**: A Go tool that statistically compares benchmark results, accounting for variance.
- **Warm-up**: PGO benefits are most visible after the binary has warmed up (JIT-like optimizations are baked in at compile time, but caches and branch predictors need warm-up).

## Real World Context
Not every service benefits equally from PGO. An I/O-bound service waiting on database queries may see < 1% improvement, while a CPU-bound JSON serialization service might see 10-14%. Measuring tells you where the investment pays off.

## Deep Dive

### Setting Up A/B Benchmarks

Build two binaries from the same source:

```bash
# Baseline: no PGO
go build -pgo=off -o server-baseline ./cmd/server

# PGO-enabled
go build -o server-pgo ./cmd/server  # Uses default.pgo
```

### Using Go Benchmarks with benchstat

```bash
# Run benchmarks 10 times each
go test -bench=. -count=10 -pgo=off ./... > nopgo.txt
go test -bench=. -count=10 ./... > pgo.txt

# Compare statistically
benchstat nopgo.txt pgo.txt
```

Example output:
```
name          old time/op  new time/op  delta
ParseJSON-8  125µs ± 2%   112µs ± 1%  -10.40%  (p=0.000 n=10+10)
ServeHTTP-8  89µs ± 3%    82µs ± 2%   -7.87%   (p=0.000 n=10+10)
```

### What PGO Optimizes

PGO focuses on three main optimizations:

1. **Inlining**: Functions that are hot in the profile are inlined more aggressively, even if they exceed normal size limits.
2. **Devirtualization**: Interface method calls where the profile shows a single dominant concrete type are converted to direct calls.
3. **Block layout**: Code blocks are reordered so the common execution path is linear (better CPU branch prediction).

### Checking Which Optimizations Applied

```bash
# See PGO-driven inlining decisions
go build -gcflags='-m -m' -pgo=default.pgo ./cmd/server 2>&1 | grep 'PGO'
```

## Common Pitfalls
1. **Running too few benchmark iterations** — Run at least 10 iterations (`-count=10`) for statistical significance. Use `benchstat` to verify the difference is real.
2. **Benchmarking I/O-bound code** — PGO optimizes CPU-bound code paths. If your bottleneck is database or network I/O, PGO won't help there.

## Best Practices
1. **Use benchstat for comparison** — Never eyeball benchmark results. benchstat accounts for variance and reports confidence intervals.
2. **Benchmark on the same hardware** — CPU architecture, cache sizes, and memory speed all affect results. Compare on identical machines.

## Summary
- Always measure PGO impact with A/B benchmarks using `-pgo=off` as baseline.
- Use `benchstat` for statistically rigorous comparison.
- PGO optimizes inlining, devirtualization, and block layout.
- Benefits are greatest for CPU-bound workloads (2-14% typical).
- Use `-gcflags='-m -m'` to see which PGO optimizations the compiler applied.

## Code Examples

**Rigorous A/B benchmark comparison — benchstat reports whether the PGO improvement is statistically significant**

```bash
# Install benchstat
go install golang.org/x/perf/cmd/benchstat@latest

# Run benchmarks with and without PGO (10 iterations each)
go test -bench=BenchmarkServe -count=10 -pgo=off ./... > baseline.txt
go test -bench=BenchmarkServe -count=10 ./... > withpgo.txt

# Statistical comparison
benchstat baseline.txt withpgo.txt
# Output shows delta with confidence intervals
```


## Resources

- [benchstat Tool](https://pkg.go.dev/golang.org/x/perf/cmd/benchstat) — Statistical benchmark comparison tool for Go

---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*