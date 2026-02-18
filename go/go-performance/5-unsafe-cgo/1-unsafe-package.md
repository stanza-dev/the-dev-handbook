---
source_course: "go-performance"
source_lesson: "go-performance-unsafe-package"
---

# unsafe.Pointer

Go is memory safe. `unsafe` allows you to bypass this safety to read/write arbitrary memory.

## Basic Conversions

```go
i := 10
p := unsafe.Pointer(&i)
f := (*float64)(p) // Reinterpret cast (dangerous!)
```

## Zero-Copy String to Bytes

```go
func StringToBytes(s string) []byte {
    return unsafe.Slice(unsafe.StringData(s), len(s))
}

func BytesToString(b []byte) string {
    return unsafe.String(&b[0], len(b))
}
```

**Warning:** Modifying the returned []byte corrupts the original string!

## Rules for unsafe.Pointer

1.  A pointer to any type can be converted to unsafe.Pointer.
2.  unsafe.Pointer can be converted to a pointer to any type.
3.  uintptr can be converted to/from unsafe.Pointer (for arithmetic).
4.  Never store uintptr in a variable (GC doesn't track it).

## Code Examples

**Unsafe String Conversion**

```go
// Zero-copy string to bytes (Go 1.20+)
func fastBytes(s string) []byte {
    return unsafe.Slice(unsafe.StringData(s), len(s))
}

// Warning: Do not modify the returned slice!
```


---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*