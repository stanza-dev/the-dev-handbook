---
source_course: "go"
source_lesson: "go-slices-basics"
---

# Slices Deep Dive

## Introduction

Slices are Go's answer to dynamic arrays—flexible, powerful, and ubiquitous. They provide a view into an underlying array without copying data, making them efficient for passing around large datasets. Understanding slice internals is crucial for writing performant, bug-free Go code.

## Key Concepts

**Slice Header**: A small struct containing a pointer, length, and capacity—not the data itself.

**Length (len)**: The number of elements the slice currently contains.

**Capacity (cap)**: The maximum number of elements before reallocation is needed.

**Backing Array**: The actual storage where slice elements live.

## Real World Context

Slices are used everywhere: function parameters, return values, JSON parsing results, database rows, file contents. Understanding that slices share underlying arrays prevents data corruption bugs. Knowing when capacity triggers reallocation helps optimize performance-critical code.

## Deep Dive

### Slice Internals

A slice header is 24 bytes on 64-bit systems:

```go
type slice struct {
    ptr *T   // Pointer to first element
    len int  // Current length
    cap int  // Total capacity
}
```

### Creating Slices

```go
// Literal (allocates underlying array)
s1 := []int{1, 2, 3}

// From array (no allocation)
arr := [5]int{1, 2, 3, 4, 5}
s2 := arr[1:4]  // [2, 3, 4], shares arr's memory

// With make
s3 := make([]int, 3)      // len=3, cap=3
s4 := make([]int, 3, 10)  // len=3, cap=10

// Nil slice (valid, len=0, cap=0)
var s5 []int  // nil
```

### Slicing Syntax

```go
s := []int{0, 1, 2, 3, 4, 5}

s[1:4]   // [1 2 3]      - elements 1, 2, 3
s[:3]    // [0 1 2]      - first 3 elements
s[3:]    // [3 4 5]      - from index 3 to end
s[:]     // [0 1 2 3 4 5] - entire slice
s[1:4:5] // [1 2 3], cap=4 - full slice expression
```

### Understanding len vs cap

```go
s := make([]int, 3, 5)
fmt.Println(len(s), cap(s))  // 3, 5

s = append(s, 4, 5)  // No reallocation (cap=5)
fmt.Println(len(s), cap(s))  // 5, 5

s = append(s, 6)  // Triggers reallocation!
fmt.Println(len(s), cap(s))  // 6, 10 (doubled)
```

## Common Pitfalls

1. **Nil vs empty slice**: `var s []int` (nil) and `s := []int{}` (empty) behave the same for most operations but have different nil checks.

2. **Capacity limits slicing**: You can only slice up to capacity, not beyond. `s[0:cap(s)]` is valid, `s[0:cap(s)+1]` panics.

3. **Append may or may not reallocate**: Depending on capacity, `append` either mutates the backing array or creates a new one.

## Best Practices

- Use `make([]T, 0, n)` when you know approximate final size.
- Prefer `nil` slices over empty slices (`var s []int` vs `s := []int{}`).
- Use `len(s) == 0` to check emptiness (works for both nil and empty).
- Remember: slices are references to arrays, not copies.

## Summary

Slices are views into backing arrays with a pointer, length, and capacity. They're created via literals, slicing arrays, or `make()`. The `append` function grows slices, potentially reallocating when capacity is exceeded. Understanding this memory model prevents bugs and enables performance optimization.

## Code Examples

**Creating and Appending to Slices**

```go
package main

import "fmt"

func main() {
    // Create a slice with make(type, len, cap)
    s := make([]string, 3)
    s[0] = "a"
    s[1] = "b"
    s[2] = "c"
    
    s = append(s, "d")
    s = append(s, "e", "f")
    
    fmt.Println(s) // [a b c d e f]
}
```


## Resources

- [Go Slices: usage and internals](https://go.dev/blog/slices-intro) — Official blog post on slice internals
- [A Tour of Go - Slices](https://go.dev/tour/moretypes/7) — Interactive slice tutorial

---

> 📘 *This lesson is part of the [Go Programming](https://stanza.dev/courses/go) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*