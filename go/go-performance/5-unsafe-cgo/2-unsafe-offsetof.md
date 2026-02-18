---
source_course: "go-performance"
source_lesson: "go-performance-unsafe-offsetof"
---

# Inspecting Memory Layout

## unsafe.Sizeof

Returns the size of a value in bytes:

```go
unsafe.Sizeof(int64(0))   // 8
unsafe.Sizeof("hello")    // 16 (string header)
unsafe.Sizeof([]int{})    // 24 (slice header)
```

## unsafe.Alignof

Returns alignment requirement:

```go
unsafe.Alignof(int64(0))  // 8
unsafe.Alignof(int32(0))  // 4
unsafe.Alignof(bool(false))  // 1
```

## unsafe.Offsetof

Returns byte offset of a struct field:

```go
type S struct {
    A int32
    B int64
}

unsafe.Offsetof(S{}.A)  // 0
unsafe.Offsetof(S{}.B)  // 8 (not 4, due to alignment)
```

## Practical Use

Calculating padding and optimizing struct layout.

## Code Examples

**Struct Layout Analysis**

```go
type Example struct {
    A bool
    B int64
    C bool
}

func printLayout() {
    fmt.Printf("Size: %d\n", unsafe.Sizeof(Example{}))
    fmt.Printf("A offset: %d\n", unsafe.Offsetof(Example{}.A))
    fmt.Printf("B offset: %d\n", unsafe.Offsetof(Example{}.B))
    fmt.Printf("C offset: %d\n", unsafe.Offsetof(Example{}.C))
}
```


---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*