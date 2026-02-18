---
source_course: "go-performance"
source_lesson: "go-performance-struct-padding"
---

# CPU Cache Lines

CPUs read memory in chunks (cache lines, usually 64 bytes). The layout of your struct affects how much memory it uses due to **alignment padding**.

```go
// Bad: 24 bytes (on 64-bit)
type Bad struct {
    A bool  // 1 byte + 7 padding
    B int64 // 8 bytes
    C bool  // 1 byte + 7 padding
}

// Good: 16 bytes
type Good struct {
    B int64 // 8 bytes
    A bool  // 1 byte
    C bool  // 1 byte + 6 padding
}
```

## Rule

Order fields from largest to smallest to minimize padding.

## Checking Size

```go
import "unsafe"

fmt.Println(unsafe.Sizeof(Bad{}))   // 24
fmt.Println(unsafe.Sizeof(Good{}))  // 16
```

## Code Examples

**Checking Size**

```go
import "unsafe"

fmt.Println(unsafe.Sizeof(Bad{}))  // Check struct size
fmt.Println(unsafe.Alignof(int64(0)))  // Check alignment requirement
```


---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*