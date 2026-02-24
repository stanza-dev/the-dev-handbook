---
source_course: "go-performance"
source_lesson: "go-performance-allocation-strategies"
---

# Allocation Reduction Strategies

## Introduction
The fastest allocation is the one that never happens. Beyond escape analysis and sync.Pool, Go offers several techniques to reduce heap allocations: pre-allocation, slice reuse, string/byte conversions, and careful API design. These techniques compound — a 50% reduction in allocations can mean a 2x improvement in throughput.

## Key Concepts
- **Pre-allocation**: Using `make([]T, 0, expectedCap)` to avoid repeated slice growth.
- **Slice reuse**: Resetting a slice's length to zero (`s = s[:0]`) while keeping its underlying array.
- **strings.Builder**: An efficient, allocation-friendly way to build strings incrementally.
- **Struct embedding**: Embedding values instead of pointers avoids pointer chasing and may reduce escapes.

## Real World Context
An API server parsing JSON bodies, validating fields, and building responses can easily make 20+ allocations per request. By pre-allocating slices, reusing buffers, and avoiding unnecessary string conversions, you can reduce that to 3-5 allocations — cutting GC overhead and improving p99 latency under load.

## Deep Dive

### Pre-allocate Slices and Maps

When you know the approximate size, pre-allocate:

```go
// Bad: grows 3 times (len 0→4→8→16) for 10 items
items := []string{}
for _, v := range data {
    items = append(items, v.Name)
}

// Good: single allocation for the final size
items := make([]string, 0, len(data))
for _, v := range data {
    items = append(items, v.Name)
}
```

For maps, use `make(map[K]V, expectedSize)` to avoid rehashing.

### Reuse Slices with [:0]

```go
// Reuse the same buffer across calls
func (p *Processor) Process(data []byte) {
    p.buf = p.buf[:0]            // Reset length, keep capacity
    p.buf = append(p.buf, data...) // Reuse existing memory
}
```

### Efficient String Building

```go
// Bad: O(n²) — creates a new string on every concatenation
result := ""
for _, s := range parts {
    result += s
}

// Good: O(n) — writes to an internal buffer
var b strings.Builder
b.Grow(estimatedSize) // Optional: pre-allocate
for _, s := range parts {
    b.WriteString(s)
}
result := b.String()
```

### Avoid Unnecessary Pointer Fields

```go
// More allocations — each field is a separate heap object
type Config struct {
    Name    *string
    Timeout *time.Duration
}

// Fewer allocations — all data is inline
type Config struct {
    Name    string
    Timeout time.Duration
    HasName bool // Use a flag for "optional" semantics
}
```

## Common Pitfalls
1. **Growing slices in a loop** — Every `append` past capacity allocates a new backing array. Pre-allocate with `make([]T, 0, n)`.
2. **String concatenation in loops** — Use `strings.Builder` or `bytes.Buffer` instead of `+=`.
3. **Converting between []byte and string repeatedly** — Each conversion allocates. Keep data in one form throughout the pipeline.

## Best Practices
1. **Profile allocations with `-benchmem`** — Run `go test -bench=. -benchmem` to see allocations per operation. Aim for zero allocations in hot paths.
2. **Use the `allocs` benchmark metric** — `b.ReportAllocs()` in benchmarks shows allocation counts.

## Summary
- Pre-allocate slices and maps when you know the expected size.
- Reuse slices with `s[:0]` to avoid repeated allocations.
- Use `strings.Builder` for efficient string construction.
- Avoid unnecessary pointer fields that force heap allocations.
- Profile with `-benchmem` and aim for zero allocations in hot paths.

## Code Examples

**Benchmarking string concatenation vs strings.Builder — Builder is O(n) with 1 allocation, concat is O(n²) with n allocations**

```go
func BenchmarkConcat(b *testing.B) {
	parts := []string{"hello", " ", "world", "!"}

	b.Run("string-concat", func(b *testing.B) {
		for i := 0; i < b.N; i++ {
			s := ""
			for _, p := range parts {
				s += p // Allocates on every iteration
			}
		}
	})

	b.Run("strings-builder", func(b *testing.B) {
		for i := 0; i < b.N; i++ {
			var sb strings.Builder
			for _, p := range parts {
				sb.WriteString(p) // Single allocation
			}
			_ = sb.String()
		}
	})
}
```


## Resources

- [Effective Go: Allocation](https://go.dev/doc/effective_go#allocation_new) — Official guidance on allocation patterns with new and make

---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*