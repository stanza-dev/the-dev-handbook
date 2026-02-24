---
source_course: "go-performance"
source_lesson: "go-performance-struct-padding"
---

# Struct Layout & Padding

## Introduction
The Go compiler adds invisible padding bytes between struct fields to satisfy CPU alignment requirements. A poorly ordered struct can waste 30-50% of memory to padding. Understanding alignment rules lets you pack structs tightly, improving cache utilization and reducing memory footprint.

## Key Concepts
- **Alignment**: CPU requirement that a value's memory address must be a multiple of its size (e.g., `int64` must be at address divisible by 8).
- **Padding**: Invisible bytes the compiler inserts between fields to satisfy alignment.
- **Struct size**: The total size including padding. Use `unsafe.Sizeof()` to check.
- **Field ordering**: Rearranging fields from largest to smallest minimizes padding.

## Real World Context
A service storing millions of user sessions in memory can save gigabytes by reordering struct fields. When each session struct drops from 72 bytes to 48 bytes (by eliminating padding), 10 million sessions save 228 MB. This also means fewer cache misses and less GC work.

## Deep Dive

### How Padding Works

Consider this struct:

```go
type Bad struct {
    Active bool    // 1 byte + 7 bytes padding (next field needs 8-byte alignment)
    ID     int64   // 8 bytes
    Count  int32   // 4 bytes + 4 bytes padding (struct aligned to largest field)
}
// Total: 24 bytes (but only 13 bytes of data!)
```

The compiler adds 7 bytes after `Active` so `ID` starts at an 8-byte boundary, and 4 bytes after `Count` so the struct size is a multiple of 8.

### Optimal Field Ordering

Sort fields from largest to smallest alignment:

```go
type Good struct {
    ID     int64   // 8 bytes
    Count  int32   // 4 bytes
    Active bool    // 1 byte + 3 bytes padding (struct must be multiple of 8)
}
// Total: 16 bytes — saved 8 bytes (33% reduction)
```

### Checking Struct Size

```go
fmt.Println(unsafe.Sizeof(Bad{}))  // 24
fmt.Println(unsafe.Sizeof(Good{})) // 16
```

### The General Rule

Order fields from largest alignment to smallest:
1. `int64`, `float64`, `unsafe.Pointer`, pointers, slices, strings (8 bytes)
2. `int32`, `float32` (4 bytes)
3. `int16` (2 bytes)
4. `bool`, `byte`, `int8` (1 byte)

### Using fieldalignment Tool

```bash
go install golang.org/x/tools/go/analysis/passes/fieldalignment/cmd/fieldalignment@latest
fieldalignment -fix ./...
```

This tool automatically reorders struct fields for optimal alignment.

## Common Pitfalls
1. **Ignoring padding in high-volume structs** — Padding waste compounds with volume. A 8-byte waste × 10M objects = 80 MB wasted.
2. **Optimizing struct layout for rarely-created types** — Focus on structs that are allocated in bulk (slice elements, map values, cache entries).

## Best Practices
1. **Run `fieldalignment` on data-heavy packages** — It automatically identifies and fixes suboptimal struct layouts.
2. **Group fields accessed together** — Beyond padding, placing frequently co-accessed fields adjacent improves cache line utilization.

## Summary
- Padding bytes are added between struct fields for CPU alignment.
- Ordering fields from largest to smallest minimizes padding waste.
- Use `unsafe.Sizeof()` to check struct sizes and `fieldalignment` to auto-fix.
- Focus optimization on structs allocated in high volumes.

## Code Examples

**Same fields, different order — 33% size reduction by ordering fields from largest to smallest alignment**

```go
package main

import (
	"fmt"
	"unsafe"
)

// Poorly ordered — wastes 8 bytes on padding
type UserBad struct {
	Active   bool   // 1 + 7 padding
	ID       int64  // 8
	Age      int32  // 4 + 4 padding
}
// Size: 24 bytes (11 bytes wasted)

// Optimally ordered — minimal padding
type UserGood struct {
	ID       int64  // 8
	Age      int32  // 4
	Active   bool   // 1 + 3 padding
}
// Size: 16 bytes (3 bytes wasted)

func main() {
	fmt.Println("Bad:", unsafe.Sizeof(UserBad{}))   // 24
	fmt.Println("Good:", unsafe.Sizeof(UserGood{})) // 16
}
```


## Resources

- [fieldalignment Tool](https://pkg.go.dev/golang.org/x/tools/go/analysis/passes/fieldalignment) — Static analysis tool that detects and fixes suboptimal struct field ordering
- [unsafe.Sizeof Documentation](https://pkg.go.dev/unsafe#Sizeof) — API reference for checking the size of types including padding

---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*