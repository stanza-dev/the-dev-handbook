---
source_course: "go-performance"
source_lesson: "go-performance-allocation-strategies"
---

# Reducing Allocations

## Pre-allocate Slices

```go
// Bad: Multiple allocations during growth
var result []int
for i := 0; i < 1000; i++ {
    result = append(result, i)
}

// Good: Single allocation
result := make([]int, 0, 1000)
for i := 0; i < 1000; i++ {
    result = append(result, i)
}
```

## Avoid String Concatenation

```go
// Bad: Creates many strings
s := ""
for _, word := range words {
    s += word
}

// Good: Single allocation
var sb strings.Builder
for _, word := range words {
    sb.WriteString(word)
}
s := sb.String()
```

## Pass by Pointer for Large Structs

```go
// Copies 1KB on each call
func process(data BigStruct) {}

// No copy
func process(data *BigStruct) {}
```

## Use Arrays Instead of Slices

```go
// Slice header escapes, array is allocated on heap
func makeSlice() []byte { return make([]byte, 32) }

// Array stays on stack
func makeArray() [32]byte { return [32]byte{} }
```

## Code Examples

**Benchmark Allocations**

```go
// Measure allocations in benchmarks
func BenchmarkAlloc(b *testing.B) {
    b.ReportAllocs()
    for i := 0; i < b.N; i++ {
        _ = make([]byte, 1024)
    }
}
```


---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*