---
source_course: "go-performance"
source_lesson: "go-performance-pgo-internals"
---

# How PGO Works

## Profile Information Used

*   **Hot functions:** Functions that consume most CPU time.
*   **Call frequencies:** How often function A calls function B.
*   **Branch probabilities:** Which branches are taken more often.

## Compiler Optimizations

### Inlining

Hot functions are more aggressively inlined:

```go
// Without PGO: May not inline
func process(i Item) { ... }

// With PGO: If hot, compiler inlines it
for _, item := range items {
    process(item)  // Becomes inline code
}
```

### Devirtualization

```go
var handler Handler = &ConcreteHandler{}
handler.Handle(data)  // Interface call

// With PGO, if ConcreteHandler is always used:
handler.(*ConcreteHandler).Handle(data)  // Direct call
```

### Block Ordering

Hot code paths are laid out linearly for better CPU cache usage.

## Code Examples

**Checking PGO Status**

```go
// Check if PGO is enabled
func main() {
    if debug.ReadBuildInfo() != nil {
        // Check for -pgo flag in build info
    }
}
```


---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*