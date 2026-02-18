---
source_course: "go"
source_lesson: "go-arrays-basics"
---

# Arrays in Go

## Introduction

Arrays are the simplest collection type in Go—a fixed-size sequence of elements of the same type. While slices (covered next) are far more common, understanding arrays is essential because slices are built on top of them. Arrays also have unique value semantics that differ from most programming languages.

## Key Concepts

**Fixed Size**: Array length is part of the type. `[5]int` and `[10]int` are different types.

**Value Semantics**: Arrays are values, not references. Assignment copies all elements.

**Contiguous Memory**: Elements are stored sequentially, enabling efficient CPU cache usage.

**Zero Initialization**: Uninitialized arrays have all elements set to zero values.

## Real World Context

Arrays appear in cryptographic code (fixed-size hashes), protocol implementations (fixed-size headers), and performance-critical code where allocation overhead matters. In most application code, you'll use slices instead, but arrays shine when the size is truly fixed and known at compile time.

## Deep Dive

### Declaration

```go
// Explicit size
var a [5]int
a[0] = 1

// Initialize with values
b := [3]string{"Go", "is", "fun"}

// Let compiler count elements
c := [...]int{1, 2, 3, 4, 5}  // Type is [5]int

// Initialize specific indices
d := [5]int{1: 10, 3: 30}  // [0, 10, 0, 30, 0]
```

### Value Semantics

```go
a := [3]int{1, 2, 3}
b := a       // b is a COPY of a
b[0] = 99
fmt.Println(a)  // [1 2 3] - unchanged
fmt.Println(b)  // [99 2 3]
```

### Function Parameters

```go
// Receives a copy (expensive for large arrays)
func process(arr [1000]int) { ... }

// Use pointer to avoid copy
func processPtr(arr *[1000]int) { ... }
```

### Comparison

```go
a := [3]int{1, 2, 3}
b := [3]int{1, 2, 3}
fmt.Println(a == b)  // true - arrays are comparable
```

## Common Pitfalls

1. **Unexpected copying**: Passing large arrays to functions copies all elements. Use pointers or slices for large data.

2. **Size is part of the type**: `[5]int` and `[10]int` are incompatible types. You can't assign one to the other.

3. **Out of bounds access**: Accessing beyond array bounds causes a panic at runtime (compile-time check when index is constant).

## Best Practices

- Prefer slices for general-purpose collections.
- Use arrays when size is fixed and known (e.g., SHA256 hash is always 32 bytes).
- Use `[...]` syntax to let the compiler count initializer elements.
- Pass large arrays by pointer to avoid expensive copies.

## Summary

Arrays are fixed-size, value-type collections where the size is part of the type. Assignment copies all elements. Arrays are comparable with `==` and initialized to zero values. While slices are more common, arrays provide value semantics and compile-time size guarantees.

## Code Examples

**Array Value Semantics**

```go
package main

import "fmt"

func main() {
    // Arrays are values
    a := [3]int{1, 2, 3}
    b := a  // Copy!
    b[0] = 99
    
    fmt.Println(a) // [1 2 3]
    fmt.Println(b) // [99 2 3]
}
```


## Resources

- [A Tour of Go - Arrays](https://go.dev/tour/moretypes/6) — Interactive introduction to arrays
- [Go Blog - Arrays, slices, and strings](https://go.dev/blog/slices) — Deep dive into slice mechanics

---

> 📘 *This lesson is part of the [Go Programming](https://stanza.dev/courses/go) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*