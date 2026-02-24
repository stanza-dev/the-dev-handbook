---
source_course: "go-performance"
source_lesson: "go-performance-unsafe-offsetof"
---

# Advanced Unsafe Patterns

## Introduction
Beyond basic type punning, `unsafe` enables several powerful patterns: struct serialization without reflection, MMIO (memory-mapped I/O), and custom memory pools. These patterns trade safety for performance in the most demanding scenarios — typically 5-20x faster than their safe equivalents.

## Key Concepts
- **Type punning**: Reinterpreting a value's bytes as a different type without conversion.
- **unsafe.Add**: Adds a byte offset to a pointer — the safe way to do pointer arithmetic in Go 1.17+.
- **unsafe.Slice**: Creates a slice from a pointer and length — avoids manual SliceHeader manipulation.
- **Reflection-free serialization**: Reading struct bytes directly, bypassing `encoding/binary` and `reflect`.

## Real World Context
High-performance databases (Badger, Pebble) use unsafe to read/write records directly from memory-mapped files. Network protocol libraries use unsafe to cast byte buffers directly to struct pointers, avoiding per-field deserialization. These patterns can process millions of records per second.

## Deep Dive

### Direct Struct Serialization

Instead of encoding field-by-field, cast the entire struct to bytes:

```go
type Header struct {
    Magic   uint32
    Version uint16
    Length  uint32
}

func headerToBytes(h *Header) []byte {
    size := unsafe.Sizeof(Header{})
    return unsafe.Slice((*byte)(unsafe.Pointer(h)), size)
}

func bytesToHeader(b []byte) *Header {
    return (*Header)(unsafe.Pointer(unsafe.SliceData(b)))
}
```

This is 10-20x faster than `encoding/binary.Read` because it avoids reflection entirely.

### Accessing Unexported Fields

unsafe can access private fields via offset (for testing and debugging):

```go
type secretStruct struct {
    public  int
    private string  // unexported
}

s := secretStruct{public: 1, private: "hidden"}
// Access private field via offset
ptr := unsafe.Add(unsafe.Pointer(&s), unsafe.Offsetof(s.private))
val := *(*string)(ptr)
fmt.Println(val) // "hidden"
```

**Warning**: This breaks encapsulation and should only be used for debugging/testing.

### Memory-Mapped I/O

```go
// Map a file into memory
data, _ := syscall.Mmap(fd, 0, size, syscall.PROT_READ, syscall.MAP_SHARED)

// Cast to a slice of structs — zero-copy
records := unsafe.Slice((*Record)(unsafe.Pointer(unsafe.SliceData(data))), numRecords)

// Access records directly from the memory-mapped file
fmt.Println(records[0].Name)
```

### The unsafe.Pointer Safety Rules

Go specifies exactly six valid `unsafe.Pointer` conversion patterns. All other patterns are undefined behavior:

1. Convert `*T` to `unsafe.Pointer` to `*U` (type punning)
2. Convert `unsafe.Pointer` to `uintptr` and back **in a single expression**
3. `unsafe.Add(ptr, offset)` for pointer arithmetic
4. `reflect.Value.Pointer()` or `.UnsafeAddr()` to `unsafe.Pointer`
5. `reflect.SliceHeader` / `reflect.StringHeader` conversions (deprecated — use `unsafe.Slice`/`unsafe.String`)
6. Passing `unsafe.Pointer` to `syscall.Syscall`

## Common Pitfalls
1. **Endianness assumptions** — Direct struct casting assumes the CPU's byte order. This breaks on big-endian architectures if you don't account for it.
2. **Struct padding differences across architectures** — A struct may have different padding on 32-bit vs 64-bit systems. Use explicit fixed-size types for serialization.

## Best Practices
1. **Add build tags for architecture constraints** — If your unsafe code assumes 64-bit or little-endian, use `//go:build amd64 || arm64`.
2. **Write comprehensive tests** — Unsafe code bypasses the type checker. Tests are your only safety net. Test on multiple architectures if possible.

## Summary
- Direct struct serialization via unsafe is 10-20x faster than reflection-based encoding.
- `unsafe.Add` and `unsafe.Slice` are the modern, safer patterns for pointer arithmetic.
- Memory-mapped I/O combined with unsafe enables zero-copy data access.
- Follow Go's six valid `unsafe.Pointer` conversion patterns — all others are undefined behavior.
- Add architecture build tags and comprehensive tests for unsafe code.

## Code Examples

**Zero-copy struct serialization — casts between struct and bytes directly, avoiding per-field encoding**

```go
// Direct struct serialization — 10-20x faster than encoding/binary
type Packet struct {
	Type   uint8
	Length uint32
	Seq    uint64
}

func encodePacket(p *Packet) []byte {
	size := unsafe.Sizeof(Packet{})
	// Cast struct pointer to byte slice — zero copy
	return unsafe.Slice((*byte)(unsafe.Pointer(p)), size)
}

func decodePacket(b []byte) *Packet {
	// Cast byte slice to struct pointer — zero copy
	return (*Packet)(unsafe.Pointer(unsafe.SliceData(b)))
}
```


## Resources

- [unsafe.Pointer Safety Rules](https://pkg.go.dev/unsafe#Pointer) — Official documentation on the six valid unsafe.Pointer conversion patterns

---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*