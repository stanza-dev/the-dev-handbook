---
source_course: "go-performance"
source_lesson: "go-performance-unsafe-package"
---

# The unsafe Package

## Introduction
Go is a memory-safe language by design — but sometimes you need to bypass safety for performance. The `unsafe` package provides direct memory access, pointer arithmetic, and type punning. It is the foundation of many high-performance Go libraries, but misuse leads to crashes, data corruption, and undefined behavior.

## Key Concepts
- **unsafe.Pointer**: A special pointer type that can be converted to/from any pointer type. It bypasses Go's type system.
- **unsafe.Sizeof**: Returns the size of a type in bytes, including padding.
- **unsafe.Alignof**: Returns the alignment requirement of a type.
- **unsafe.Offsetof**: Returns the byte offset of a struct field from the struct's start.
- **uintptr**: An integer type large enough to hold any pointer. Used for pointer arithmetic.

## Real World Context
The Go standard library uses `unsafe` extensively for performance: `reflect`, `sync`, `runtime`, `strings`, and `encoding/binary` all use unsafe internally. High-performance libraries like `fasthttp`, `sonic` (fast JSON), and `goccy/go-json` rely on unsafe for zero-copy string/byte conversions and direct struct manipulation.

## Deep Dive

### unsafe.Pointer Conversions

The fundamental unsafe operation is pointer type conversion:

```go
// Convert *int to *float64 — reinterprets the bytes
var i int64 = 42
f := *(*float64)(unsafe.Pointer(&i))
```

Go's rules for `unsafe.Pointer` conversions:
1. Any pointer → `unsafe.Pointer` ✓
2. `unsafe.Pointer` → any pointer ✓
3. `uintptr` → `unsafe.Pointer` (only in specific patterns)
4. Pointer arithmetic via `uintptr` (only in specific patterns)

### Struct Field Access by Offset

```go
type Config struct {
    Name    string
    Port    int
    Debug   bool
}

c := Config{Name: "app", Port: 8080, Debug: true}

// Access Port field directly via offset
portPtr := (*int)(unsafe.Add(unsafe.Pointer(&c), unsafe.Offsetof(c.Port)))
fmt.Println(*portPtr) // 8080
```

`unsafe.Add` (Go 1.17+) is the safe way to do pointer arithmetic — it's a single expression that the compiler can verify.

### Zero-Copy String/Byte Conversion

The most common performance use of unsafe — converting between `string` and `[]byte` without copying:

```go
// Zero-copy string to []byte (READ-ONLY — never modify the result!)
func stringToBytes(s string) []byte {
    return unsafe.Slice(unsafe.StringData(s), len(s))
}

// Zero-copy []byte to string
func bytesToString(b []byte) string {
    return unsafe.String(unsafe.SliceData(b), len(b))
}
```

These avoid the ~50-100 ns copy for large strings/byte slices.

## Common Pitfalls
1. **Modifying bytes from a zero-copy string conversion** — Strings in Go are immutable. Modifying the bytes behind a string causes undefined behavior and may crash.
2. **Storing uintptr values** — A uintptr is just an integer; the GC doesn't know it points to anything. The object may be collected while you hold the uintptr.
3. **Breaking across GC cycles** — Never convert `unsafe.Pointer` → `uintptr` and back in separate statements. The GC may move the object between the two operations.

## Best Practices
1. **Prefer unsafe.Add and unsafe.Slice** — These Go 1.17+ functions provide safer patterns than raw pointer arithmetic.
2. **Isolate unsafe code** — Keep unsafe operations in small, well-tested functions with clear contracts about safety requirements.

## Summary
- `unsafe.Pointer` enables type punning and pointer arithmetic, bypassing Go's type system.
- `unsafe.Sizeof`, `Alignof`, `Offsetof` inspect type memory layout.
- Zero-copy string/byte conversion avoids allocation but requires immutability discipline.
- Use `unsafe.Add` and `unsafe.Slice` (Go 1.17+) for safer pointer arithmetic.
- Isolate unsafe code in small functions with clear safety contracts.

## Code Examples

**Zero-copy conversions using Go 1.17+ unsafe functions — StringData, SliceData, and unsafe.String avoid allocations**

```go
package main

import (
	"fmt"
	"unsafe"
)

// Zero-copy []byte to string — avoids allocation
func bytesToString(b []byte) string {
	return unsafe.String(unsafe.SliceData(b), len(b))
}

// Zero-copy string to []byte — result is READ-ONLY
func stringToBytes(s string) []byte {
	return unsafe.Slice(unsafe.StringData(s), len(s))
}

func main() {
	data := []byte("hello world")
	s := bytesToString(data)  // No copy, no allocation
	fmt.Println(s)            // "hello world"

	fmt.Println(unsafe.Sizeof(int64(0)))  // 8
	fmt.Println(unsafe.Alignof(int64(0))) // 8
}
```


## Resources

- [unsafe Package](https://pkg.go.dev/unsafe) — Official API reference for Go's unsafe package including safety rules

---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*