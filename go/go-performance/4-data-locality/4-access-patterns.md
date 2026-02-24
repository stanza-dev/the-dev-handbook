---
source_course: "go-performance"
source_lesson: "go-performance-memory-access-patterns"
---

# Memory Access Patterns & SIMD

## Introduction
Beyond data layout, the *pattern* of memory access matters enormously. Sequential access is fast because the CPU's hardware prefetcher predicts and preloads the next cache line. Random access defeats the prefetcher. Go 1.26 introduces the `simd` package for explicit SIMD (Single Instruction, Multiple Data) operations, letting you process multiple data elements in a single CPU instruction.

## Key Concepts
- **Sequential access**: Reading/writing memory addresses in order (arrays, slices). The CPU prefetcher loads the next cache line before you need it.
- **Random access**: Reading/writing memory at unpredictable addresses (hash maps, pointer chasing). Each access is a likely cache miss.
- **Prefetcher**: CPU hardware that detects sequential access patterns and loads upcoming cache lines in advance.
- **SIMD (Single Instruction, Multiple Data)**: CPU instructions that process 4-16 values simultaneously. Go 1.26 adds the experimental `simd/archsimd` package (amd64 only, requires `GOEXPERIMENT=simd`).

## Real World Context
Image processing libraries process pixels sequentially — loading one cache line of pixel data processes 8 pixels at once. Combined with SIMD, a single instruction can transform 4-8 pixels simultaneously. The combination of sequential access + SIMD can yield 10-50x speedups over naive random-access approaches.

## Deep Dive

### Sequential vs Random Access

```go
// Sequential: ~2 ns per access (prefetcher helps)
data := make([]int, 1_000_000)
for i := range data {
    data[i]++
}

// Random: ~20-50 ns per access (cache misses)
indices := rand.Perm(len(data))
for _, i := range indices {
    data[i]++
}
```

The random version is 10-25x slower because every access misses the cache and the prefetcher cannot predict the next address.

### Binary Search vs Linear Scan

For small datasets (< 512 elements), linear scan can beat binary search because of sequential access patterns:

```go
// Binary search: O(log n) but random access patterns
sort.SearchInts(data, target)

// Linear scan: O(n) but sequential access — faster for n < 512
for _, v := range data {
    if v == target { break }
}
```

### SIMD in Go 1.26 (Experimental)

Go 1.26 introduces the `simd/archsimd` package as an **experimental** feature, gated behind `GOEXPERIMENT=simd`. Currently it supports **amd64 only** with 128-bit, 256-bit, and 512-bit vector types:

```go
// Requires: GOEXPERIMENT=simd go build
import "simd/archsimd"

// Process 4 float64 values in a single instruction (amd64 only)
var a, b archsimd.Float64x4
a = archsimd.Float64x4{1.0, 2.0, 3.0, 4.0}
b = archsimd.Float64x4{5.0, 6.0, 7.0, 8.0}
result := a.Add(b) // {6.0, 8.0, 10.0, 12.0}
```

**Important**: This is experimental and not covered by the Go 1 compatibility promise. A high-level portable SIMD package is planned for a future release.

### When SIMD Helps

SIMD is most effective for:
- Numerical computation (vector math, matrix operations)
- Data filtering and transformation (applying the same operation to many elements)
- String/byte processing (searching, comparing)

## Common Pitfalls
1. **Assuming maps are cache-friendly** — Go maps use hash tables with pointer-based buckets. For hot-path lookups with small key sets, a sorted slice with binary search may be faster.
2. **Ignoring stride** — Accessing every Nth element of a large array (large stride) defeats the prefetcher. Keep access patterns tight.

## Best Practices
1. **Prefer sequential access in hot loops** — Iterate slices in order. Avoid index-based random access when possible.
2. **Consider SIMD for bulk operations** — When processing arrays of numbers on amd64, Go 1.26's experimental `simd/archsimd` package can provide 2-8x speedups (requires `GOEXPERIMENT=simd`).

## Summary
- Sequential memory access is 10-25x faster than random access due to CPU prefetching.
- Binary search beats linear scan for large datasets; linear scan wins for small ones.
- Go 1.26 adds `simd/archsimd` as an experimental, amd64-only SIMD package (requires `GOEXPERIMENT=simd`).
- SIMD processes 4-16 values per instruction — ideal for numerical and bulk data operations.
- Keep access patterns sequential and data contiguous for maximum cache utilization.

## Code Examples

**Experimental SIMD vector addition in Go 1.26 — requires GOEXPERIMENT=simd and only works on amd64**

```go
// Go 1.26 experimental SIMD (amd64 only)
// Build with: GOEXPERIMENT=simd go build
package main

import "simd/archsimd"

func vectorAdd(a, b []float64) []float64 {
	result := make([]float64, len(a))
	// Process 4 elements per iteration using 256-bit vectors
	for i := 0; i+4 <= len(a); i += 4 {
		va := archsimd.LoadFloat64x4(a[i:])
		vb := archsimd.LoadFloat64x4(b[i:])
		archsimd.StoreFloat64x4(result[i:], va.Add(vb))
	}
	// Handle remaining elements with scalar code
	for i := len(a) &^ 3; i < len(a); i++ {
		result[i] = a[i] + b[i]
	}
	return result
}
```


## Resources

- [Go 1.26 Release Notes](https://go.dev/doc/go1.26) — Official release notes covering the new simd and simd/archsimd packages

---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*