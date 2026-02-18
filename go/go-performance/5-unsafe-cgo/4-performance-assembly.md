---
source_course: "go-performance"
source_lesson: "go-performance-assembly"
---

# Assembly in Go

For maximum performance, you can write assembly. Go uses a portable assembly syntax.

## File Structure

```
mypkg/
  add.go       // Go stub
  add_amd64.s  // AMD64 assembly
  add_arm64.s  // ARM64 assembly
```

## Go Stub

```go
// add.go
package mypkg

//go:noescape
func Add(a, b int64) int64
```

## Assembly (AMD64)

```asm
// add_amd64.s
TEXT ·Add(SB), NOSPLIT, $0-24
    MOVQ a+0(FP), AX
    MOVQ b+8(FP), BX
    ADDQ BX, AX
    MOVQ AX, ret+16(FP)
    RET
```

## When to Use

*   SIMD operations (vectorization).
*   Cryptographic primitives.
*   Extremely hot paths.

**Note:** Most programs never need assembly. Profile first!

## Code Examples

**Assembly Stub**

```go
// Compiler directive to use assembly implementation
//go:noescape
func vectorSum(data []float64) float64

// Fallback pure Go implementation
func vectorSumGo(data []float64) float64 {
    var sum float64
    for _, v := range data {
        sum += v
    }
    return sum
}
```


---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*