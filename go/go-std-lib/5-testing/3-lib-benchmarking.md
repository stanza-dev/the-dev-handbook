---
source_course: "go-std-lib"
source_lesson: "go-std-lib-benchmarking"
---

# Performance Testing

Go has first-class support for benchmarks using `testing.B`.

```go
func BenchmarkLogic(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Logic()
    }
}
```

The tool automatically adjusts `b.N` until the function runs for long enough to get reliable statistics.

## Running Benchmarks

```bash
go test -bench=.                  # All benchmarks
go test -bench=BenchmarkLogic     # Specific benchmark
go test -bench=. -benchmem        # Include memory stats
go test -bench=. -count=5         # Multiple runs
```

## Setup Outside Loop

```go
func BenchmarkSort(b *testing.B) {
    data := generateData(10000)  // Setup
    
    b.ResetTimer()  // Don't count setup time
    
    for i := 0; i < b.N; i++ {
        sort.Ints(data)
    }
}
```

## Memory Allocations

Use `b.ReportAllocs()` to track memory allocations per operation.

## Code Examples

**Running benchmarks with memory stats**

```bash
go test -bench . -benchmem -cpuprofile=cpu.out
```


---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*