---
source_course: "go-performance"
source_lesson: "go-performance-cgo-basics"
---

# CGO: Calling C from Go

## Introduction
CGO lets Go programs call C functions and use C libraries. This is essential when you need to use a C library with no Go equivalent (SQLite, OpenSSL, system APIs). However, CGO has significant overhead per call due to stack switching and scheduler coordination. Go 1.26 reduced this overhead by ~30%, but it remains significant for fine-grained calls. Understanding this cost helps you decide when CGO is worthwhile.

## Key Concepts
- **CGO**: Go's foreign function interface for calling C code from Go.
- **`import "C"`**: A pseudo-package that enables CGO. C declarations in the comment immediately above this import become accessible in Go.
- **CGO call overhead**: Each CGO call has overhead from stack switching, signal handling, and scheduler coordination. Go 1.26 reduced this overhead by ~30% compared to earlier versions.
- **`//go:nosplit`**: A compiler directive that prevents stack growth checks — used in performance-critical code but dangerous if the function uses too much stack.

## Real World Context
SQLite, libsodium, and many image/video processing libraries are C-only. CGO lets Go programs use these battle-tested libraries. However, calling a C function that takes 5 ns to execute incurs tens of nanoseconds in overhead — a significant penalty. For such cases, batching calls or using Go-native alternatives is better.

## Deep Dive

### Basic CGO Usage

```go
package main

/*
#include <math.h>
*/
import "C"
import "fmt"

func main() {
    result := C.sqrt(C.double(144.0))
    fmt.Println(float64(result)) // 12.0
}
```

The comment block before `import "C"` is the "preamble" — it can contain `#include`, `#define`, and even inline C functions.

### CGO Call Overhead Breakdown

Each CGO call involves:
1. Switching from Go's segmented stack to a C-compatible stack
2. Saving/restoring signal handlers
3. Notifying the Go scheduler that the goroutine is in a syscall
4. Performing type conversions and argument marshaling

Go 1.26 reduced this baseline overhead by ~30% through optimized stack switching. The total overhead is typically tens of nanoseconds per call — far more than a Go function call (~2 ns) but far less than a process context switch.

### Batching to Amortize Overhead

Instead of calling C once per item, pass a batch:

```go
/*
// Process an array in C — one call instead of N
void process_batch(double* data, int len) {
    for (int i = 0; i < len; i++) {
        data[i] = sqrt(data[i]);
    }
}
*/
import "C"

func processBatch(data []float64) {
    C.process_batch((*C.double)(unsafe.Pointer(&data[0])), C.int(len(data)))
}
```

One CGO call for 1000 items incurs the overhead once, vs 1000× the overhead for individual calls.

### CGO Build Requirements

CGO requires a C compiler. Common gotchas:

```bash
# Disable CGO for cross-compilation or minimal builds
CGO_ENABLED=0 go build .

# CGO is enabled by default when a C compiler is available
CGO_ENABLED=1 go build .
```

## Common Pitfalls
1. **Calling C in a tight loop** — The per-call overhead dominates for trivial C functions. Batch operations or move the loop to C.
2. **Passing Go pointers to C** — Go's GC may move objects. Use `C.malloc` for C-owned memory, or ensure Go pointers follow the CGO pointer-passing rules.
3. **Memory management confusion** — C allocations (`C.malloc`) must be freed with `C.free`. Go's GC will not collect them.

## Best Practices
1. **Batch CGO calls** — Pass arrays/slices to C instead of calling per-element. Amortize the per-call overhead across many operations.
2. **Consider pure Go alternatives** — If a Go-native library exists (e.g., `modernc.org/sqlite` for SQLite), it avoids CGO overhead entirely.

## Summary
- CGO enables calling C functions from Go via the `import "C"` pseudo-package.
- Each CGO call has significant overhead (tens of nanoseconds). Go 1.26 reduced it by ~30%.
- Batch operations to amortize overhead — one call for N items, not N calls.
- C memory (`C.malloc`) must be manually freed with `C.free`.
- Consider pure Go alternatives to avoid CGO overhead entirely.

## Code Examples

**CGO example with proper memory management — C.CString allocates, C.free releases, data is copied across the boundary**

```go
package main

/*
#include <stdlib.h>
#include <string.h>

// C function: uppercase a string in-place
void to_upper(char* s, int len) {
    for (int i = 0; i < len; i++) {
        if (s[i] >= 'a' && s[i] <= 'z') {
            s[i] -= 32;
        }
    }
}
*/
import "C"
import (
	"fmt"
	"unsafe"
)

func toUpper(s string) string {
	// Allocate C memory, copy string, process, copy back
	cs := C.CString(s)         // Go string → C string (copies)
	defer C.free(unsafe.Pointer(cs)) // Must free C memory!

	C.to_upper(cs, C.int(len(s)))
	return C.GoString(cs)      // C string → Go string (copies)
}

func main() {
	fmt.Println(toUpper("hello")) // HELLO
}
```


## Resources

- [CGO Documentation](https://pkg.go.dev/cmd/cgo) — Official CGO documentation covering calling conventions and pointer rules
- [CGO Wiki](https://go.dev/wiki/cgo) — Community wiki with CGO best practices and common patterns

---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*