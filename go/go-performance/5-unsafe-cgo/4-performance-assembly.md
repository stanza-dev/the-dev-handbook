---
source_course: "go-performance"
source_lesson: "go-performance-assembly"
---

# Go Assembly & Compiler Intrinsics

## Introduction
When every nanosecond counts, Go supports hand-written assembly for specific architectures. Go assembly uses a unique Plan 9-derived syntax that is portable across operating systems but architecture-specific. While rarely needed, assembly is used in the standard library for cryptography, math, and runtime internals.

## Key Concepts
- **Go assembly**: Plan 9-style assembly, written in `.s` files alongside Go code.
- **Compiler intrinsics**: Built-in functions the compiler translates directly to specific instructions (e.g., `math/bits.OnesCount` → `POPCNT`).
- **`//go:noescape`**: A directive that tells the compiler a function's pointer arguments don't escape, allowing the caller to stack-allocate them.
- **`//go:nosplit`**: Prevents stack growth checks — saves ~1 ns per call but risks stack overflow if the function is too deep.

## Real World Context
The Go standard library uses assembly for `crypto/aes`, `crypto/sha256`, `math/big`, and parts of the runtime (scheduler, memcpy). These are the 0.1% of functions where assembly makes a measurable difference. For the other 99.9%, the Go compiler produces excellent code.

## Deep Dive

### Compiler Intrinsics — Assembly Without .s Files

Many performance-critical operations are available as compiler intrinsics:

```go
import "math/bits"

// Compiled to a single POPCNT instruction
count := bits.OnesCount64(x)

// Compiled to a single LZCNT/CLZ instruction
leading := bits.LeadingZeros64(x)

// Compiled to a single TZCNT/CTZ instruction
trailing := bits.TrailingZeros64(x)
```

These are faster than manual bit manipulation and more readable than assembly.

### Writing Go Assembly

Go assembly files (`.s`) sit alongside Go files. The Go file declares the function signature; the `.s` file provides the implementation:

```go
// math_amd64.go
//go:noescape
func addVectors(a, b, result *float64, n int)
```

```asm
// math_amd64.s
TEXT ·addVectors(SB), NOSPLIT, $0-32
    MOVQ a+0(FP), SI
    MOVQ b+8(FP), DI
    MOVQ result+16(FP), DX
    MOVQ n+24(FP), CX
loop:
    MOVSD (SI), X0
    ADDSD (DI), X0
    MOVSD X0, (DX)
    ADDQ $8, SI
    ADDQ $8, DI
    ADDQ $8, DX
    DECQ CX
    JNZ loop
    RET
```

### Compiler Directives for Performance

```go
//go:noescape
func fastHash(data []byte) uint64

//go:nosplit
func criticalPath() {
    // No stack growth check — saves ~1 ns
}

//go:noinline
func mustNotInline() {
    // Prevents inlining — useful for benchmarks
}
```

### When to Use Assembly vs SIMD Package

Go 1.26 introduced the experimental `simd/archsimd` package (amd64 only, `GOEXPERIMENT=simd`), which can be an alternative to hand-written assembly for numerical work:

| Feature | Go Assembly | simd Package |
|---------|-------------|--------------|
| Portability | Architecture-specific | Architecture-specific (experimental) |
| Maintainability | Hard to read/modify | Readable Go code |
| Performance | Maximum possible | Near-optimal |
| Use case | Crypto, math primitives | Bulk numerical processing (amd64) |

## Common Pitfalls
1. **Writing assembly when intrinsics suffice** — `math/bits` and `simd` cover most common cases. Only write assembly for operations with no Go equivalent.
2. **Forgetting architecture-specific file naming** — Assembly files must be named `*_amd64.s`, `*_arm64.s`, etc. The wrong suffix means the file is silently ignored.

## Best Practices
1. **Use compiler intrinsics first** — `math/bits`, `sync/atomic`, and `simd` compile to optimal instructions without assembly maintenance burden.
2. **Benchmark against pure Go** — The Go compiler is good. Verify your assembly or `simd/archsimd` code is actually faster before committing to maintaining it.

## Summary
- Go assembly uses Plan 9 syntax in `.s` files, paired with Go function declarations.
- Compiler intrinsics (`math/bits`) are the easiest way to get specific CPU instructions.
- `//go:noescape` and `//go:nosplit` optimize function call overhead.
- Go 1.26's experimental `simd/archsimd` package can replace hand-written SIMD assembly on amd64.
- Only write assembly when intrinsics and the `simd` package can't provide the needed performance.

## Code Examples

**Compiler intrinsics from math/bits — each function compiles to a single CPU instruction, no assembly needed**

```go
package main

import (
	"fmt"
	"math/bits"
)

func main() {
	var x uint64 = 0b1010_1100_1110_0001

	// Compiler intrinsics — compile to single CPU instructions
	fmt.Println("Population count:", bits.OnesCount64(x))   // POPCNT
	fmt.Println("Leading zeros:", bits.LeadingZeros64(x))    // LZCNT
	fmt.Println("Trailing zeros:", bits.TrailingZeros64(x))  // TZCNT
	fmt.Println("Bit length:", bits.Len64(x))                // BSR

	// Rotate operations — compile to ROL/ROR
	rotated := bits.RotateLeft64(x, 5)
	fmt.Printf("Rotated: %b\n", rotated)
}
```


## Resources

- [Go Assembly Guide](https://go.dev/doc/asm) — Official documentation on writing Go assembly and Plan 9 syntax
- [math/bits Package](https://pkg.go.dev/math/bits) — API reference for compiler intrinsics — bit manipulation functions that compile to single instructions

---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*