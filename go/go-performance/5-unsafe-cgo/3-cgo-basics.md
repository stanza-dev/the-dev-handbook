---
source_course: "go-performance"
source_lesson: "go-performance-cgo-basics"
---

# CGO

CGO allows calling C code from Go.

```go
/*
#include <stdlib.h>
#include <math.h>
*/
import "C"
import "fmt"

func main() {
    result := C.sqrt(16)
    fmt.Println(result)  // 4
}
```

## CGO Overhead

CGO calls are expensive (~100ns vs ~2ns for Go calls) due to:
*   Stack switching.
*   Scheduler interaction.
*   Memory copying for some types.

## When to Use CGO

*   Interfacing with existing C libraries.
*   Hardware access.
*   Performance-critical code in C.

## When to Avoid

*   Simple operations (overhead dominates).
*   High-frequency calls.
*   Cross-platform code (CGO is platform-specific).

## Code Examples

**CGO Math Library**

```go
/*
#cgo LDFLAGS: -lm
#include <math.h>
*/
import "C"

func sqrt(x float64) float64 {
    return float64(C.sqrt(C.double(x)))
}
```


---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*