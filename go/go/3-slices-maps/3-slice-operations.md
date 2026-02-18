---
source_course: "go"
source_lesson: "go-slice-operations"
---

# Slice Operations & Gotchas

## Introduction

Slices seem simple but hide complexity that trips up even experienced Go developers. The shared backing array behavior can cause subtle bugs, and understanding when `append` reallocates is crucial for correctness. This lesson covers essential operations and the gotchas you must know.

## Key Concepts

**Append**: Built-in function that adds elements, potentially reallocating if capacity is exceeded.

**Copy**: Built-in function that copies elements between slices, returning the count copied.

**Shared Backing Array**: Multiple slices can reference the same underlying array, meaning modifications affect all.

**Full Slice Expression**: Syntax `s[low:high:max]` that limits capacity, preventing unintended sharing.

## Real World Context

The shared backing array gotcha causes real production bugs: caching a slice, modifying it later, and corrupting the cache. Understanding these patterns is essential for concurrent code where race conditions can arise from shared slice data.

## Deep Dive

### Append Operations

```go
s := []int{1, 2}
s = append(s, 3)           // Single element
s = append(s, 4, 5, 6)     // Multiple elements
s = append(s, other...)    // Another slice (spread)

// IMPORTANT: Always reassign!
s = append(s, x)  // ✓
append(s, x)      // ✗ Result is lost!
```

### Copy Function

```go
src := []int{1, 2, 3}
dst := make([]int, len(src))
n := copy(dst, src)  // n = 3

// Copy only copies min(len(dst), len(src)) elements
small := make([]int, 2)
copy(small, src)  // Only copies [1, 2]
```

### The Shared Array Gotcha

```go
a := []int{1, 2, 3, 4, 5}
b := a[1:3]  // b = [2, 3], shares a's array

b[0] = 99
fmt.Println(a)  // [1 99 3 4 5] - a was modified!
```

### Append Can Share OR Reallocate

```go
a := make([]int, 3, 5)  // len=3, cap=5
b := append(a, 4)       // Room in capacity: shares array!
a[0] = 99
fmt.Println(b[0])       // 99 - b was affected!

// But if capacity exceeded:
c := append(b, 5, 6)    // Needs reallocation
b[0] = 0
fmt.Println(c[0])       // 99 - c has independent array
```

### Safe Slice Copy

```go
// Create independent copy
original := []int{1, 2, 3}
safeCopy := make([]int, len(original))
copy(safeCopy, original)

// Or using append
safeCopy := append([]int(nil), original...)
```

### Deleting Elements

```go
// Delete at index i
s = append(s[:i], s[i+1:]...)

// Using slices package (Go 1.21+)
import "slices"
s = slices.Delete(s, i, i+1)
```

## Common Pitfalls

1. **Forgetting to reassign append result**: `append(s, x)` without `s =` silently discards the result.

2. **Assuming append always reallocates**: When capacity suffices, append modifies the existing backing array, affecting other slices.

3. **Modifying function parameters**: Slices passed to functions share the backing array—modifications persist after return.

## Best Practices

- Always assign `append` result: `s = append(s, x)`.
- Use `copy` or `append([]T(nil), s...)` to create independent slices.
- Use full slice expression `s[low:high:high]` to limit capacity and prevent sharing.
- Use `slices.Delete` instead of manual append for clarity.

## Summary

`append` adds elements and may reallocate if capacity is exceeded. `copy` transfers elements between slices. Multiple slices can share a backing array, causing modifications to propagate unexpectedly. Use the full slice expression or explicit copying to create independent slices.

## Code Examples

**Shared Backing Array Gotcha**

```go
package main

import "fmt"

func main() {
    // Gotcha: shared backing array
    a := []int{1, 2, 3, 4, 5}
    b := a[1:3]  // b = [2, 3]
    b[0] = 99
    
    fmt.Println(a) // [1 99 3 4 5] - a was modified!
    fmt.Println(b) // [99 3]
}
```


## Resources

- [Go Wiki - Slice Tricks](https://go.dev/wiki/SliceTricks) — Common slice manipulation patterns
- [slices package](https://pkg.go.dev/slices) — Standard library slice utilities

---

> 📘 *This lesson is part of the [Go Programming](https://stanza.dev/courses/go) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*