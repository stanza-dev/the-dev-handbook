---
source_course: "go-performance"
source_lesson: "go-performance-stack-vs-heap"
---

# Stack vs Heap Allocation

## Introduction
Every variable in Go lives on either the stack or the heap. Stack allocation is nearly free — it's just moving a pointer. Heap allocation requires garbage collection later. Understanding where Go places your variables is the foundation of memory performance optimization.

## Key Concepts
- **Stack allocation**: Memory allocated on the goroutine's stack. It is automatically freed when the function returns — no GC involvement.
- **Heap allocation**: Memory allocated on the shared heap. The garbage collector must track and eventually free it.
- **Escape analysis**: The compiler's static analysis that determines whether a variable can stay on the stack or must escape to the heap.
- **Escape report**: Output from `go build -gcflags='-m'` showing which variables escape and why.

## Real World Context
A JSON serialization function that allocates a new buffer on every call can generate millions of heap allocations per second under load. Moving that buffer to the stack (or reusing it via sync.Pool) can reduce GC pressure by 10x and cut p99 latency dramatically.

## Deep Dive

### How Escape Analysis Works

The compiler analyzes each variable to determine if its lifetime extends beyond the function that created it. If it does, the variable "escapes" to the heap.

Variables escape when they:
- Are returned by pointer from a function
- Are stored in an interface value (the compiler can't always prove the size)
- Are captured by a goroutine or closure that outlives the function
- Exceed a size threshold (very large arrays/slices)

### Checking Escape Analysis Output

```bash
go build -gcflags='-m' ./...
```

Example output:
```
./main.go:10:2: moved to heap: result
./main.go:15:9: &Point{} escapes to heap
./main.go:20:2: x does not escape
```

### Stack vs Heap Performance

```go
// This stays on the stack — no allocation
func sumLocal() int {
    x := 42
    y := 58
    return x + y
}

// This escapes to the heap — the pointer outlives the function
func newPoint() *Point {
    p := Point{X: 1, Y: 2}
    return &p // p escapes because caller holds a pointer
}
```

The `sumLocal` function costs ~0 ns for allocation. The `newPoint` function triggers a heap allocation (~25-50 ns) plus future GC work.

### Go 1.26 Stack Allocation Improvements

Go 1.26 improved the compiler to allocate slice backing stores on the stack in more situations. Previously, `make([]T, n)` would almost always heap-allocate; now the compiler can keep small, locally-scoped slices on the stack when it proves they don't escape.

## Common Pitfalls
1. **Unnecessary pointer returns** — Returning `*MyStruct` when the struct is small (< ~64 bytes) forces a heap allocation. Return by value instead.
2. **Interface boxing** — Passing a value as an `interface{}` may cause it to escape. Use concrete types in hot paths.
3. **Premature optimization** — Not all heap allocations matter. Focus on allocations in hot loops, not one-time setup code.

## Best Practices
1. **Check escape analysis for hot functions** — Run `go build -gcflags='-m'` on performance-critical code and address unnecessary escapes.
2. **Prefer value receivers for small structs** — Value receivers keep the struct on the stack. Pointer receivers may cause escapes.

## Summary
- Stack allocation is nearly free; heap allocation requires GC.
- The compiler's escape analysis decides placement automatically.
- Use `go build -gcflags='-m'` to see what escapes and why.
- Common escape triggers: pointer returns, interface boxing, goroutine captures.
- Go 1.26 improved stack allocation for slice backing stores in more situations.

## Code Examples

**Value return keeps data on the stack; pointer return forces a heap allocation — verify with escape analysis flags**

```go
package main

type Point struct {
	X, Y float64
}

// Value return — Point stays on caller's stack
func newPointValue() Point {
	return Point{X: 1, Y: 2}
}

// Pointer return — Point escapes to heap
func newPointPtr() *Point {
	p := Point{X: 1, Y: 2}
	return &p // compiler: "moved to heap: p"
}

// Check with: go build -gcflags='-m' main.go
```


## Resources

- [Go FAQ: Stack or Heap](https://go.dev/doc/faq#stack_or_heap) — Official FAQ explaining how Go decides between stack and heap allocation
- [Go Memory Management Guide](https://go.dev/doc/gc-guide) — Comprehensive guide to Go's garbage collector and memory management

---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*