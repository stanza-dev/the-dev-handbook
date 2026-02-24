---
source_course: "go-std-lib"
source_lesson: "go-std-lib-benchmarking"
---

# Benchmarking

## Introduction
Optimization without measurement is guesswork. Go ships a first-class benchmarking framework in the `testing` package that lets you measure execution time and memory allocations with a single command. This lesson covers how to write benchmarks, interpret results, and avoid the measurement traps that produce misleading numbers.

## Key Concepts
- **testing.B**: The type passed to benchmark functions. It controls the iteration count (`b.N`), timer, and allocation tracking.
- **b.N**: An integer that the test runner adjusts automatically. Your benchmark loop runs `b.N` times, and the framework increases `b.N` until the total execution time is statistically stable.
- **b.ResetTimer()**: Zeroes the benchmark clock. Call this after expensive setup so that setup time is excluded from the measurement.
- **b.ReportAllocs()**: Enables per-operation memory allocation reporting in the benchmark output.

## Real World Context
You are deciding between `strings.Builder` and `fmt.Sprintf` for generating log messages in a hot path. Rather than guessing, you write two benchmarks and run `go test -bench=. -benchmem`. The output shows that `strings.Builder` is 3x faster and allocates half the memory per operation. You commit the benchmark alongside your code so that future refactors can be validated against the same baseline.

## Deep Dive
Benchmark functions live in `_test.go` files and start with `Benchmark` followed by a capitalized name. They accept a single `*testing.B` parameter.

Here is the simplest possible benchmark.

```go
func BenchmarkLogic(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Logic()
    }
}
```

The framework calls this function multiple times, increasing `b.N` each round until the total execution time is long enough (at least 1 second by default) to produce reliable statistics. You never set `b.N` yourself.

Benchmarks do not run by default. You must pass the `-bench` flag to `go test`.

```bash
go test -bench=.                  # All benchmarks
go test -bench=BenchmarkLogic     # Specific benchmark
go test -bench=. -benchmem        # Include memory stats
go test -bench=. -count=5         # Multiple runs for stability
```

The `-benchmem` flag adds two columns to the output: bytes allocated per operation and allocations per operation. The `-count` flag runs the benchmark multiple times so you can spot variance.

### Excluding Setup from Measurement

If your benchmark requires expensive setup (loading data, building structures), perform it before the loop and call `b.ResetTimer()` to exclude that time.

```go
func BenchmarkSort(b *testing.B) {
    data := generateData(10000) // Setup

    b.ResetTimer() // Don't count setup time

    for i := 0; i < b.N; i++ {
        sort.Ints(data)
    }
}
```

Without `b.ResetTimer()`, the setup time is included in the measurement, inflating the reported ns/op and making your benchmark useless for comparison.

### Tracking Memory Allocations

Call `b.ReportAllocs()` inside the benchmark to include allocation statistics even without the `-benchmem` flag.

```go
func BenchmarkConcat(b *testing.B) {
    b.ReportAllocs()
    for i := 0; i < b.N; i++ {
        _ = fmt.Sprintf("hello %s", "world")
    }
}
```

The output includes `B/op` (bytes per operation) and `allocs/op` (heap allocations per operation). Reducing allocations often has a bigger impact on performance than reducing CPU time because it reduces garbage collector pressure.

### Sub-benchmarks

Just like tests, benchmarks support `b.Run` for parameterized comparisons.

```go
func BenchmarkHash(b *testing.B) {
    sizes := []int{64, 256, 1024, 4096}
    for _, size := range sizes {
        b.Run(fmt.Sprintf("size=%d", size), func(b *testing.B) {
            data := make([]byte, size)
            b.ResetTimer()
            for i := 0; i < b.N; i++ {
                sha256.Sum256(data)
            }
        })
    }
}
```

Sub-benchmarks appear in the output with hierarchical names like `BenchmarkHash/size=64`, making it easy to compare performance across input sizes.

As of Go 1.26, benchmark functions also have access to `b.ArtifactDir()`, which returns a unique per-benchmark directory for writing profile data or output files. This pairs well with `-cpuprofile` and `-memprofile` flags for deep performance analysis.

## Common Pitfalls
1. **Doing setup inside the benchmark loop** -- Any work inside the `for i := 0; i < b.N` loop is measured. If you allocate test data inside the loop, you are benchmarking allocation plus your logic.
2. **Letting the compiler optimize away the result** -- If the result of the benchmarked function is never used, the compiler may eliminate the call entirely. Assign the result to a package-level variable: `var result int` and `result = Logic()` inside the loop.
3. **Running benchmarks on a loaded machine** -- Background processes, thermal throttling, and power management cause variance. Use `-count=5` or more and look for consistency across runs.

## Best Practices
1. **Always use -benchmem** -- Memory allocations are often the real bottleneck. Measuring only CPU time misses half the picture.
2. **Use benchstat for comparisons** -- The `golang.org/x/perf/cmd/benchstat` tool compares benchmark results across runs and reports whether differences are statistically significant.
3. **Commit benchmarks alongside code** -- Benchmarks serve as performance regression tests. When a refactor slows things down, the benchmark catches it before it ships.

## Summary
- Benchmark functions start with `Benchmark`, accept `*testing.B`, and loop `b.N` times.
- The framework automatically adjusts `b.N` for statistically stable results.
- Use `b.ResetTimer()` to exclude setup and `b.ReportAllocs()` to track memory.
- Sub-benchmarks with `b.Run` enable parameterized performance comparisons.
- Go 1.26 adds `b.ArtifactDir()` for per-benchmark output files alongside CPU and memory profiles.

## Code Examples

**Running benchmarks with memory stats**

```bash
go test -bench . -benchmem -cpuprofile=cpu.out
```


## Resources

- [testing package - Benchmarks](https://pkg.go.dev/testing#hdr-Benchmarks) — Official reference for writing and running benchmarks in Go

---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*