---
source_course: "go-performance"
source_lesson: "go-performance-stack-vs-heap"
---

# Escape Analysis

Go decides where to allocate memory automatically.
*   **Stack:** Fast, automatic cleanup when function returns.
*   **Heap:** Slower, managed by GC.

A variable **escapes to the heap** if its reference is returned from the function or passed to a function that might store it.

## Checking Decisions
Run `go build -gcflags="-m"` to see escape analysis decisions.

```text
./main.go:10:6: moved to heap: x
```

## Common Escape Causes

*   Returning a pointer to a local variable.
*   Storing in a variable that outlives the function.
*   Interface values (the concrete value often escapes).
*   Closures capturing local variables.
*   Slices larger than a threshold.

## Stack-friendly Code

```go
// Escapes (returns pointer)
func newInt() *int {
    x := 42
    return &x
}

// Doesn't escape (caller allocates)
func initInt(x *int) {
    *x = 42
}
```

## Code Examples

**Heap Escape**

```go
func createPointer() *int {
    x := 10 // escapes to heap because a pointer is returned
    return &x
}
```


---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*