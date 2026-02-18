---
source_course: "go-performance"
source_lesson: "go-performance-memory-access-patterns"
---

# Access Patterns

## Sequential vs Random Access

```go
// Fast: Sequential (cache prefetch works)
for i := 0; i < len(data); i++ {
    sum += data[i]
}

// Slow: Random (cache misses)
for _, idx := range randomIndices {
    sum += data[idx]
}
```

## Row-Major Order

```go
// 2D array access
matrix := [1000][1000]int{}

// Fast: Row-major (Go's memory layout)
for i := range matrix {
    for j := range matrix[i] {
        matrix[i][j] = 1
    }
}

// Slow: Column-major (poor cache usage)
for j := 0; j < 1000; j++ {
    for i := 0; i < 1000; i++ {
        matrix[i][j] = 1
    }
}
```

## Prefetching

Go doesn't have explicit prefetch, but sequential access enables hardware prefetching automatically.

## Code Examples

**Cache-Aware Matrix Multiply**

```go
// Matrix multiplication - cache aware
func multiply(a, b, c [][]float64, n int) {
    // Transpose b for better cache usage
    bt := transpose(b)
    
    for i := 0; i < n; i++ {
        for j := 0; j < n; j++ {
            sum := 0.0
            for k := 0; k < n; k++ {
                sum += a[i][k] * bt[j][k]  // Both sequential
            }
            c[i][j] = sum
        }
    }
}
```


---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*